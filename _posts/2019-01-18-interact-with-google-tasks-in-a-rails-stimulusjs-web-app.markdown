---
toc: true
toc_label: "Table of Contents"
toc_icon: "file-alt"
layout: single
title:  "Interact With Google Tasks In a Rails Stimulusjs Web App"
date:   2019-01-18 20:18:33 -0700
categories: ruby rails stimulusjs google-api service-object
---
Build a Rails app utilizing ``google-api`` gem to connect Google Tasks.
Replace Sprokets with Webpack and use Stimulusjs for frontend.

## Intro

A Rails 5 App connecting to Google Tasks with ``google-api`` ruby gem.
Frontend built with Webpack, Stimulus.js and bulma.io, completely replace
sprockets with webpack for assets management. Utilizing Redis to store
user credentials returned from Google Api requests for later use. It's
still WIP but can connect with Google API and display content already.

## Motivation

I use Google Tasks a lot, but its web interface is only accessible in Gmail
as a right side bar app. Why not making a stand alone App using Rails?
And I want to try to combine Stimulusjs with Typescript, see if the can
get along well.

## Demo

### Live Demo

### Screenshots

![get-cred](/assets/images/google-api-get-cred.gif){:width="600px"}
![auth](/assets/images/google-api-auth.gif){:width="600px"}
![list Google Tasks](/assets/images/google-taskslist.png){:width="400px"}

## What I've learned

### * Extract complex logic out from controller into service objects.

Rails follows the MVC pattern, as the app grows, there are 3 general
rules to keep large size apps organized and intuitive:
* Avoid fat Models, bloated Models can be shrinked with concerns, etc.
* Keep Views dumb, no complex logic there.
* Keep Controllers skinny & dry, Service Objects at your service.
It's very straight forward to work with Service Objects in Rails app.
Just create a ``app/services`` dir, and name each one of them as
``xxx_service.rb``. They can be either a regular Ruby class or module, here
is an example of Service Objects in my app
{% highlight ruby %}
# frozen_string_literal: true

require 'google/apis/tasks_v1'
require 'googleauth'
require 'googleauth/stores/redis_token_store'

class GoogleTasksService
  OOB_URI = 'urn:ietf:wg:oauth:2.0:oob'
  APPLICATION_NAME = 'Rails Stimulus Google Tasks'
  SCOPE = Google::Apis::TasksV1::AUTH_TASKS_READONLY
  ALREADY_AUTHORIZED = "Already authorized."
  CREDENTIALS_KEY_PREFIX = 'g-user-credentials-json:'
  AUTH_CODE_KEY_PREFIX = 'g-user-auth-code:'

  def initialize(credentials, username)
    @credentials = credentials
    @username = username
  end

  def get_auth_url
    credentials_hash = JSON.parse @credentials
    authorizer = create_authorizer_with(credentials_hash)
    credentials = authorizer.get_credentials(@username)
    return authorizer.get_authorization_url(base_url: OOB_URI) if credentials.nil?
    ALREADY_AUTHORIZED
  end

  # Store auth token in selected token store(eg, redis) and return it
  # @param [String] token
  # autorization token obtained from auth url generated from get_auth_url
  # @return [Google::Auth::UserRefreshCredentials]
  def get_and_store_credentials(token)
    credentials_hash = JSON.parse @credentials
    authorizer = create_authorizer_with(credentials_hash)
    credentials = authorizer.get_credentials(@username)
    if credentials.nil?
      # store credentials in redis with key like "g-user-token:@username"
      credentials = authorizer.get_and_store_credentials_from_code(
        user_id: @username, code: token, base_url: OOB_URI
      )
    end
    credentials
  end

  def get_tasklists(code)
    tasks_service = Google::Apis::TasksV1::TasksService.new
    tasks_service.client_options.application_name = APPLICATION_NAME
    tasks_service.authorization = get_and_store_credentials(code)

    response = tasks_service.list_tasklists(max_results: 10)
    response.items
  end

  # Create an instance of Google::Auth::UserAuthorizer with credentials hash
  # @param [Hash] credentials_hash
  # Google API credentials, example: {"installed":{"client_id":"...", "client_secret":"..."}}
  # @return [Google::Auth::UserAuthorizer]
  private def create_authorizer_with(credentials_hash)
    client_id = Google::Auth::ClientId.from_hash(credentials_hash)
    token_store = Google::Auth::Stores::RedisTokenStore.new(redis: $redis)
    Google::Auth::UserAuthorizer.new(client_id, SCOPE, token_store)
  end
end
{% endhighlight %}

### * How to get rid of sprockets and replace it with webpack.

Webpack is going to be the new default assets manangment tool in the up
coming rails 6. We can use Webpack to manage all your javascript,
css and images. Use foreman to fire up server and compile assets with a
Procfile works like a charm. Remove ``app/assets`` dir and put frontend
related code into ``app/frontend``, add your js & css dependencies
with yarn. Webpack entrance file is at
``app/frontend/packs/application.js`` and could be sth like this:
{% highlight javascript %}
/* eslint no-console:0 */
// This file is automatically compiled by Webpack, along with any other files
// present in this directory. You're encouraged to place your actual application logic in
// a relevant structure within app/javascript and only use these pack files to reference
// that code so it'll be compiled.
//
// To reference this file, add <%= javascript_pack_tag 'application' %> to the appropriate
// layout file, like app/views/layouts/application.html.erb

import * as ActiveStorage from "activestorage";
import Rails from 'rails-ujs';
import Turbolinks from 'turbolinks';
import { Application } from "stimulus";
import { definitionsFromContext } from "stimulus/webpack-helpers";
import 'bulma/css/bulma.css'
import "./application.css";

ActiveStorage.start();
Rails.start();
Turbolinks.start();

const application = Application.start();
const context = require.context("controllers", true, /.ts$/);
application.load(definitionsFromContext(context));
{% endhighlight %}
Note I put rails ActiveStorage, Rails-UJS and Turbolinks initialization
here as well and they are installed with yarn as well.


### * How to combine authlogic with pundit.

User sessions management with authlogic gem, user authorization using pundit.
Devise is more popular user authentication solution in Rails community,
it's easy to setup and running but always take a lot more efforts when
it comes to customization(which I've had a bad time with). Create a
UserSession model and controller:
{% highlight ruby %}
class UserSession < Authlogic::Session::Base
  # tell authlogic to find by username, email or phone_number
  # this way even with <%= f.text_field :username %> in signin form, user can
  # still login with either username, email or phone number
  # https://github.com/binarylogic/authlogic/blob/master/lib/authlogic/session/password.rb#L20
  find_by_login_method :find_by_username_email_or_phone
end

# and then use it in controller like this:
class UserSessionsController < ApplicationController
  before_action :ensure_not_already_login, only: [:new, :create]

  def new
    @user_session = UserSession.new
  end

  def create
    @user_session = UserSession.new(user_session_params.to_h)
    if @user_session.save
      flash[:notice] = "Login successfully!"
      redirect_back_or root_path
    else
      redirect_to login_path,
                  alert: "A problem's occured while logging in, please try again."
    end
  end

  def destroy
    if helpers.current_user_session.destroy
      redirect_to root_path, notice: "Log out successfully, see you soon!"
    end
  end

  private def user_session_params
    params.require(:user_session).permit(:username, :password, :remember_me)
  end
end

# and its associated helper module
module ApplicationHelper
  def current_user_session
    return @current_user_session if defined?(@current_user_session)
    @current_user_session = UserSession.find
  end
  ...
end
{% endhighlight %}

### * How to use Google API gem in a rails app to access Google Tasks.

There are two options when storing returned credential tokens from Google,
``FileTokenStore`` or ``RedisTokenStore``. I use the 2nd one in this
app. Refer to ``GoogleTasksService`` class above for more details. Google
also have very good documentations for how to use it.

### * How to create frontend using Stimulus.js with Typescript.

Stimulus.js IMO is a good replacement for jquery and it works pretty
well with Turbolinks and Rails-UJS. Basecamp claimed it as a modest
JS library and use it in production for a while. It doesn't require all
the client side state management like those in React.js or Vue.js. It
uses plain old html ``data`` attribute to do its jobs. However it's not
for everyone and every app, it's still a solid option for many apps IMHO.
It's like party of the Rails Way to me, please see my another post about
Rails way and frontend-backend seperation. Typescript's build in Type
check is very good way to avoid bugs in javascript code, makes it more
intuitive and easier to test, debug and maintain. Here's an example Stimulusjs
controller.
{% highlight javascript %}
import { Controller } from "stimulus";
import FlashHelper from "../utils/flashHelper";

class HomeController extends Controller {
  static targets = ["credentials", "hiddenCredentials", "authCode"];

  private credentialsTarget: HTMLInputElement;
  private hiddenCredentialsTarget: HTMLInputElement;
  private authCodeTarget: HTMLInputElement;

  connect() {
    document.body.addEventListener("ajax:success", this.onXHRSuccess);
    document.body.addEventListener("ajax:error", this.onXHRError);
  }

  disconnect() {
    document.body.removeEventListener("ajax:success", this.onXHRSuccess);
    document.body.removeEventListener("ajax:error", this.onXHRError);
  }

  urlFormSubmit(e: Event) {
    if (!this.credentialsTarget.value) {
      e.preventDefault();
      const flash = new FlashHelper("Credentials cannot be empty!", "warning");
      flash.display();
    } else {
      this.hiddenCredentialsTarget.value = this.credentialsTarget.value;
    }
  }

  authFormSubmit(e: Event) {
    if (!this.authCodeTarget.value) {
      e.preventDefault();
      const flash = new FlashHelper(
        "Authentication code cannot be empty!",
        "warning"
      );
      flash.display();
    }
  }

  // Rails-ujs event handlers, arrow function binds this to current context
  private onXHRSuccess = (event: CustomEvent) => {
    const res = event.detail[0];
    if (res.is_display_notification) {
      const flash = new FlashHelper(
        res.notification.content,
        res.notification.type
      );
      flash.display();
    }
    if (res.is_manipulate_dom) {
      this.updateContentFor(res.selector, res.content, res.is_authorized);
    }
  };

  private onXHRError = () => {
    const flash = new FlashHelper(
      "Something went wrong please try again",
      "danger"
    );
    flash.display();
  };

  // TODO make this more generic
  private updateContentFor(
    selector: string,
    content: string,
    isAuthorized: boolean
  ) {
    const target: HTMLAnchorElement = document.querySelector(selector);
    if (isAuthorized) {
      target.textContent = content; // Already authorized
    } else {
      target.href = content;
      target.textContent = "Get the authorization code";
    }
  }
}

export default HomeController;
{% endhighlight %}
