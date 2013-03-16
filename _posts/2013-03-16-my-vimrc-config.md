---
layout: post
title: "My vimrc config"
description: ""
category: quick tips
tags: [vi]
---
{% include JB/setup %}

## Settings

    set fileformat=unix
    set encoding=utf-8

    set tabstop=4
    set shiftwidth=4
    set softtabstop=4

    set expandtab

    syntax on
    color koehler

    autocmd BufEnter * :syntax sync fromstart

    set nu

Very Python-centric I must say.