---
layout: post
title: "Auth Using Username Instead Of E-mail in Laravel"
description: ""
category: quick tips
tags: [php, laravel]
---
{% include JB/setup %}


From the loginUsername definition in **Illuminate\Foundation\Auth\AuthenticatesUsers**,

    public function loginUsername() {
        return property_exists($this, 'username') ? $this->username : 'email';
    }

we can see that it tries to look for "username" first, failing which it will use "email" to identify the user.

Therefore, by renaming "name" to "username" in *create_users_table* migration file, and adding

    protected $username = 'username';

in AuthController, the authentication will be done using username instead of e-mail.


## Reference

* [Modify existing Authorization module (email to username) - Stack Overflow](http://stackoverflow.com/a/31387345)