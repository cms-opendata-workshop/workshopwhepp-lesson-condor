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
$ ssh userXX@ui3.indiacms.res.in
~~~
{: .language-bash}

After entering the password, you should see the following on your screen:
~~~
FIXME -- TIFR splash screen
~~~
{: .output}

**Great!** You're now set up for the rest of the lesson on January 10. 

{% include links.md %}

