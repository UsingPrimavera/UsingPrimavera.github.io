---
layout: post
title: "building a unifier business process"
description: "building a unifier business process"
date: 2022-11-19T12:03:47+00:00
author: barrie
categories:
image:
mermaid: true
---
This is the first of a series of posts about building a Unifier business process.  The final business process we want to build will be an application helpdesk process.  The process looks like this:
<pre class="mermaid">
sequenceDiagram
    Client->>Support Operator: New Client Request
    activate Support Operator
    opt Client request needs clarification
        loop agree understanding
            Support Operator->>Client: Further information required
            Client->>Support Operator: Updated Client Request
        end
    end
    Support Operator->>Support Consultant: Clarified Client request
    deactivate Support Operator
</pre>