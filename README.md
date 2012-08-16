`authlogic_device_tokens`
=========================

This is a plugin for integrating user device tokens into authlogic. A given user can have 1 or more devices. Every decice will have an auth token (to handle rails server user authentication) as well as a socket token. You can completely ignore the socket token if you don't need it or use it for authentication against a sockets server.

This plugin is based on [`authlogic_facebook_shim`](https://github.com/james2m/authlogic_facebook_shim)

How it works
------------

This will create a session and persist it through a cookie for the given auth_token

    UserSession.create(:auth_token => params[:auth_token])

You can know if the user was logged with a token and get the device asking the session:

    session.logged_in_with_device?
    session.device

If you want to invalidate any (non-tokenized) current session before creating a new one with a token, you can use the following. This will make sure only one session is valid at the moment, invalidating the old session and removing the proper cookie.

    UserSession.destroy_if_any
    UserSession.create(:auth_token => params[:auth_token])
    
and viceversa:

    UserSession.destroy_if_any :device  # here we are making sure to invalidate only device cookies
    UserSession.create(...)


Configuration
-------------

There are a few things you can configure in your model. Just do this:

    class User < ActiveRecord::Base
     acts_as_authentic do |c|
      c.device_cookie_key = symbol  # name of the key in the cookies hash when authenticating by device
      c.device_session_key = symbol  # name of session key when authenticating by device
      c.device_auth_token_field = symbol  # device DB field to be used for authentification
      c.device_socket_token_field = symbol  # device DB field to be used for sockets authentification
      c.device_finder = symbol or string  # what method should be used to find the Device given a token?
      c.user_device_finder = symbol or string  # What method should be used to find the User?
     end
    end

Leaving everything as default (i.e. not configuring any of these fields) will require the following requirements in your Rails application:

First and more important you will need to create a new DB table for devices with the following minimun fields (you can extend it as much as you want):

    create_table "devices" do |t|
      t.string   "auth_token"
      t.string   "socket_token"
      t.integer  "user_id"
    end

In the User model you will need the following (modify according to your setup):

    has_many :devices, :dependent => :destroy
    
    def self.find_by_device_auth_token token
      Device.find_by_auth_token(token, :joins => [:user => :account]).nil_or.user
    end


Copyright
---------

Copyright (c) 2012 Jesus Laiz (aka zheileman). See LICENSE for details.
