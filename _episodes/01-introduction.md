---
title: "Check access to TIFR (Jan 5)"
teaching: 0
exercises: 0
questions:
- "Can you log in to TIFR to use condor?"
objectives:
- "Learn the workshop login commands for TIFR and test your access."
keypoints:
- "Logging in to the TIFR cluster is easy for workshop participants!"
---

## Log in with ssh
For this lesson, many temporary accounts have been created on a TIFR computing cluster.
The local facilitators will provide you with a **username** and **password** to use for this activity.
If you are following this lesson apart from the local group at WHEPP, please reach out [on the Mattermost channel](https://mattermost.web.cern.ch/cmsodwswhepp24/channels/pre-exercises-help) to request login credentials. 

In a terminal on your local computer (**NOT** inside a docker container), use `ssh` to connect to the cluster:
~~~
$ ssh userXX@ui3.indiacms.res.in  # replace XX with the number provided to you
~~~
{: .language-bash}

After entering the password, you should see the following on your screen:
~~~
bob@localpc:wheep2024$ ssh user1@ui3.indiacms.res.in
user1@ui3.indiacms.res.in's password: 
Last login: Wed Jan  3 00:32:46 2024 from 14.139.98.164
[user1@ui3 ~]$ 
~~~
{: .output}

## Discussion prompts

- How much storage space do these TIFR accounts have?
- How long will this TIFR account be active?
- How can I reach out for support?

**Great!** You're now set up for the rest of the lesson on January 10. 

{% include links.md %}

