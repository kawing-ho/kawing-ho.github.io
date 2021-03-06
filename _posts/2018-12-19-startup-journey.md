---
layout: post
title: Summer internship at a startup
date: 2018-12-09
categories: ['operations', 'job']
---

> I did a casual internship over summer at a lovely place called **STEMN**!   
  I worked under [<u>Jackson Delahunt</u>](https://github.com/sabrehagen){:target="_blank"} with two other interns from UNSW too

<div class="divider"></div>
### How was it like?
This was my first time working at a **startup**,  
therefore I got to experience the pros and cons that came with it ------  
<br/>

:heavy_check_mark:  
- Wearing whatever you want  
- Working on literally the couch or kitchen table or the floor if you want  

- Super flexible working hours and/or presence in office  
  (I usually avoid remote work though as I get easily distracted at home)

- Home office has an awesome rooftop and two cats :cat2:  
  (unfortunately terrible if you have allergies)  

- Occasional treats such as tea or ice creams to boost morale :tea:

- Very flat hierarchy  
  (can share memes around and sometimes watch educational videos on the TV)   

___

:x:  
- The payroll was usually late :money_with_wings:   
  (luckily I had a cushion to accomodate for delays)


- No solid goals or deadlines  
  (can be a good or bad thing depending on how you look at it I suppose _??_)

<div class="divider"></div>

### What were my contributions?  

> There wasn't really a fixed set of requirements for my role, more of you just handle whatever is thrown at you, and I was told I did that pretty well :wink:   

<br/>

#### Sec
Had a chance to apply what I had learnt in **`COMP6843`** by performing a week-long penetration test of the `dev` STEMN web application   
- Tested most of the **OWASP Top Ten Vulnerabilities** ranging from `XSS` to `SQLi` but found nothing exploitable in that timespan  

<br/>

As part of a _**<u>Chaos Monkeying</u>**_ exercise I also fuzzed API endpoints :see_no_evil:  
-  discovered faulty logic that would crash the backend when certain requests were made    
 
 <br/>

While white-box testing (I was given credentials to the remote GCP servers) I also found unsecured cloud-metadata directories and alerted Jackson to the risks involved and proposed a fix

<br/>

___

#### DevOps

Learnt from documentation and created pre-baked :hotsprings: machine images for the remote development-environment using [**`Packer`**](https://www.packer.io/){:target="_blank"}.
  - Pre-baked in this context refers to creating the machine images in which server instances are built upon from

  - It is useful for populating a server instance with binaries/executables and files that you need but know won't be updated much in the future

  - For files with constant changes such as `Git` repositories it is better to clone them during provision-time (during the creation of the instance)

  - Worth mentioning that it is a bad idea to bake your **secrets** into images  

<br/>

Experimented around with `Vagrant` and `Terraform` , definitely learnt a lot more about **Infrastructure as Code**.  
  - (in fact I made my own Terraform [mini-project](https://github.com/kawing-ho/terraocean){:target="_blank"} !)

<br/>

Learned a bit about **`Kubernetes`** --- (`kubectl` commands and `kubeconfig` files) but wouldn't consider enough to say _<u>"I know how to use Kubernetes"</u>_ :ship:   
  - I think the autoscaling feature is really neat as it can
    - scale up to handle a sudden increase in traffic
    - scale down (even to zero) during idle traffic times
  - Had a chance to brush up on `Dockerfile` knowledge for a bit too 

<br/>

___

#### Tooling/Coding


I was assigned the task of writing a **`yarn shim`**/task runner because the original `yarn` was running too slow



Here is the task running _"tree"_ that shows the parent-child relationships and how some processes have to wait for others to finish before proceeding   

|||
| ![draft](/assets/images/startup-draft.jpg) | ![draft](/assets/images/startup-draft-overlay.jpg) |  

<br/>

##### Functionality  (three parts)
1) A shim or initial waypoint that can
  - call itself recursively
  - delegate to the original `yarn`

<br/>

2) The original `yarn`
  - It is a package manager similar to `npm`
    - in this context it serves as a task runner    
    - ie.  a form of aliasing using [scripts](https://yarnpkg.com/lang/en/docs/cli/run/){:target="_blank"} but they can be stacked together  

<br/>

3) [**`Jaeger`**](https://www.jaegertracing.io/){:target="_blank"} tracing  
  - it allows you to track progress across processes and even remote hosts!

<br/>
<br/>

##### Timeline

1) <u>Reverse Engineered the precompiled 'lib' version of yarn</u>
  - Found hook-able locations to do debugging and logging
  - ie the _reporter_ function  

<br/>
2) <u>Make ascii-style logs which uses post-processing</u>
  - The result trace data is not realtime but rather produced at the end
    - bad if process hangs, data would never be collected  
  - example log:  ![ascii-log](/assets/images/startup-ascii.jpg)

<br/>

3) <u>Made changes to the forked repo and recompiled</u>
  - Interestingly `console.log` would break the chain of scripts
    - Some commands rely on the output of others to function
    - Therefore had to use `console.error` instead :thinking:   

<br/>


4) <u>Introduced Jaeger tracing, tweaked around to work</u>  
  - Works really well for time-series data, but also works for timing here
  - Inject/Extract context from env vars to set them under one trace
    
  - example Jaeger UI, notice the hierarchy on the left is not working:  ![yarn-initial](/assets/images/startup-yarn-1.png)

<br/>

5) <u>Wrote the shim, tested to make sure it works</u>  
  - Got massive amounts of help from Jackson as I was completely new to `TS`

<br/>

6) <u>Dropped logging in the shim, added tracing instead</u>

<br/>

7) <u>Wrote and testing the linking/unlinking of shim over original yarn</u>

<br/>

8) <u>Finally got hierarchy to work</u> :100: 
  - Had to figure out the logic using `childOf` and `followsFrom` in context
  - Also some counterintuitive methods like updating context **AFTER** the process has spawned etc

  - final example: ![yarn-final](/assets/images/startup-yarn-2.png) 

<br/>

Overall this whole process took roughly over two weeks but I am very proud about _(hopefully)_ leaving my mark in STEMN history with this tool!  :smile:  

<br/>

___

#### Misc
Became a guinea pig for new integrations into the workflow :hamster: ie.  
  - trying out snap docker versus regular docker
  - `kubeadm` versus `micok8s`
  - `Riot` Matrix app versus `Slack`
  - random stuff like `D3 library`, `Jaeger`

<br/>

Using [**`Hub`**](https://hub.github.com/){:target="_blank"} by _GitHub_ to issue pull requests via CLI
  - requires you to create a personal access token first 
  - really handy for those moments where you just pushed your changes and are too lazy to open the site on the browser
  - **Hub** also supports other functions such as forking and creating issues

<br/>


Played around with a command-line IRC client known as [**`WeeChat`**](https://weechat.org/about/){:target="_blank"} 
- spent time tweaking the UI such as
  - hiding join/part messages
  - adjusting width/colours of sidebars
- their customization is done by editing a file during runtime and the changes are applied during runtime as well

- wrote a startup script to automate the joining of IRC channels as well 
  - ie. _#nodejs_, _#ubuntu_ etc...
- oh yeah it's written in C by the way  :thinking:

<br/>

[**`Husky`**](https://github.com/typicode/husky){:target="_blank"} for Git Hooking
  - pre-commit hooks can be used to restrict illegal actions 
    - if a user is trying to commit to a restricted branch ie. `staging`
  - post-push hooks can print GitHub URL pointing to that specific commit 
    - useful for quickly sharing/reviewing changes with the _diff_ view 

<br/>

Also wrote a **`"git split"`** script
- allows you to choose which changes to be commited to the current branch
- leftover/undesired changes would be stored in a new _split-off_ branch :construction:  

- essentially just a wrapper _Shell_ script that does the following:  

   ```bash
    git status

    # prompt user for input/ notify user
    # .... stuff .... 

    git add --interactive

    # prompts for branch name
    # ...
    # prompts for name of split-off branch
    # ... 
    # pushes both
   ```  

<div class="divider"></div>

### What else did I learn?

<br/>

#### Javascript 
- **Node.js** basics (in real-world applications)  
  - Understanding syntax and conventions
      - Arrow function syntax/nameless functions were confusing at first  
  - How `Promises` and `await/async` can prevent _callback hell_

- **Jest** testing 
  - `--watch` is very useful! :eyes:  
  - `describe()`, `it()` and _mocking_ are interesting as well
  - the only similar testing I used previously was Django's unit testing

- **Functional Javascript** vs Regular JS
- **Typescript** as a superset of JS

- Package ecosystems and package managers such as `yarn` and `npm` :bento: 
  - using lock files to freeze the version of a dependency you need for other developers to use to stay consistent

  - different projects can share or have their own local dependency installed too

<br/>

#### Development
- **Kubernetes**
  - hardening and attacking 
  - the whole notion of pods, nodes, clusters, contexts, etc  
  - self-signed certificates and Role Based Access Control (RBAC)
  - quite cool how `kubectl` given a context file can communicate with a local cluster or a secured cluster in the open
  - `Helm` --- like `apt` but for Kubernetes stuff

- Lifecycle of **cloud application development**
  - continuous integrations
  - operations using _Hashicorp_ utilities

- Proper **dev netiquette** :squirrel:  
  - raising _Issues_ in public repositories
  - seeking help in IRC channels

<br/>

#### Misc
- git aliases ie. `git s` ---> `git status`  :octocat:

- tmux
  - [`tmuxinator`](https://github.com/tmuxinator/tmuxinator#tmuxinator){:target="_blank"} --- really awesome way to pre-load your working setup 
  - more advanced tmux usage
    - tmux shortcuts and window management
    - detached sessions and tmux server
    - nesting remote tmux inside local tmux
    - useful tmux plugins such as [<u>URLViewer</u>](https://github.com/tmux-plugins/tmux-urlview){:target="_blank"} for opening links in tmux
- The importance of platform agnostic scripts
  - wrote a linking/unlinking solution for the shim above
    - actually broke on Apple OSX because `whereis` doesn't exist on OSX
    - in the end `hash` was used as the check and replace method :apple: 

- **`VSCode`**
  - why yes, I am a `Vim` user who had to learn how to use `VSCode`! ![relevant_meme](/assets/images/startup-meme.png)

<br/>

#### Knowledge exchange

- Stuff that Jackson learnt from me:
  - asciinema 
  - bash syntax such as `!$` --- used to get args of previous command
  - rtv _(reddit terminal viewer)_  
  - apropos 

- He keeps me up-to-date with bleeding-edge tech such as `Istio`, `Kubernetes Serverless` and others 

- He containerized his entire working environment, so it can be pulled and run from prety much anywhere! 

- He also set up his home desktop to become available from any web browser: 
  - using a mixture of `gotty` and `Web Hook Relay`  
  - mounting the X-server and getting re-renders (ie. window resizing) to work was the only difficult part

  - right now he is relying on security through obscurity but I will advise him to change that soon :secret:

- Cloud Native Foundation and Knative
<div class="divider"></div>

### April 2019 Update

During the summer I used to come in almost every weekday.  
Recently to balance with my other jobs at UNSW I only come in 2 days a week  
The workload has slowed down significantly as well...

<br/>
Since the very beginning of my internship I told Jackson I wouldn't stay for long as I wanted to progress down the security route and this job wouldn't exactly do it for me, so I am now parting ways with **STEMN**,  
all the best to them in the future! :wind_chime:   


> Overall really awesome learning experience, I learnt not just technical knowledge but also heaps of life stuff as well ~   
(such as spiritualism, productivity etc...) :tanabata_tree:   


