---
layout: post
title: How to run process in background?
permalink: blog/run-process-in-background-with-screen/
comments: True
excerpt_separator: <!--more-->
---

NOTE: You should not use this workflow in production, but for development and quick tests, this is fine.

Now that we have that notice out of the way, let's do this! 

- run `screen`
- enjoy the alternate dimension. Here, will be able to run your process without using `nohup` and `&`
- run your process, for example: `node server.js`
- now, detach from screen with **ctrl+a, d**. You will not cause a SIGINT to the process and you'll go back to the "real world". 
- note that your process is still running inside a screen session
- list sessions you are detached from with `screen -ls`
- reattach to one with `screen -r [sessionid]`

There you go! You can also use `tmux` if you want, which is similar but have more features and better user experience. The proper way to do this in production however is to use something like `pm2` or `supervisord` or creating a startup script.
