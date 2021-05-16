---
layout: post
title:  "Secret Santa"
date:   2020-12-21 17:02:00 +0000
categories: algorithms
---

It seems that the end of 2020 is going to look very much like the rest of the year. Like most other Londoners, the last minute lockdown announcements have put the final nail in the coffin of my holiday plans, which means I get to spend, *once again*, some more time at home. This is the opportunity to reflect on this strange year. While I have not ticked all the boxes in my original 2020 wish list (no half marathon, no summer trip to Italy), this has been the opportunity to accomplish other things, sometimes planned and sometimes unexpected. I bought a house, passed my ABSRM piano grade 1 exam, learnt some new recipes, renewed with a long lost love for console games, read some CS books, started this blog.

So how about a bit of a festive touch for the last entry of the year? In this post I'll talk about a small script that I put together to solve a very concrete problem that I actually had, which is a nice change from most of the coding I do.

# A seasonal problem

Most people who celebrate Christmas in large groups will be familiar with the concept of Secret Santa. Let's say there are N guests at the party, rather than having everyone buying N-1 gifts and ending up with lots of thoughtless, average, and possibly duplicated presents, each person is secretly assigned one other person that they will buy a present for. Less pre-Christmas stress, more thoughtful presents, happier people.

Sometimes the context calls for additional constraints, like exclusion rules. In my family for instance, we usually organise a secret santa between my uncles, aunts and cousins, but everyone also typically give presents to their immediate family members (parents, siblings, etc.) regardless of the secret santa draw. However we don't want the secret santa draw to overlap with the latter, so we would like to "exclude" immediate family members from the list of "possible secret santa giftees" for each participant.

There are plenty of tools online that can take care of the drawing for you and distribute the results to all participants by email. You enter everyone's email address, configure the exclusion rules, click a big flashy button and boom, your drawing is done. In the process, you have also earned yourself and your family a lifetime of spam, and everyone's email address has probably been sold to a dozen phishers by the time you check your inbox.

That's why this year I decided to go for a more DIY approach, and wrote my own script to handle the draw. 

# A simple solution

Like so many other problems in the world, this can be represented and solved with graphs. Each participant is a vertex, there is a directed edge from each participant to each of the other participants that the person is allowed to draw. Then all you need to do is a simple graph travesal: start from a random vertex, mark it visited, pick a random neighbour, repeat.

The resulting traversal sequence is your draw chain. If you traversed in order A -> B -> C -> D, then A is getting a present for B, B is getting a present for C, etc. 

For the purpose of the demo I have used Game of Thrones characters. I have set exclusions on direct relationships, including and limited to: parent/child (biological or adopted), sibling, first cousin, uncle/aunt/nephew/niece.

```python
#!/usr/bin/env python

import random
import smtplib, ssl

class SecretSantaDraw:
    # Adjacency list
    graph = {'Ned':  ['Daenerys', 'Cersei', 'Jamie', 'Hodor', 'Drogon', 'Night King'],
             'Arya':  ['Daenerys', 'Cersei', 'Jamie', 'Hodor', 'Drogon', 'Night King'],
             'Sansa': ['Daenerys', 'Cersei', 'Jamie', 'Hodor', 'Drogon', 'Night King'],
             'Jon':  ['Cersei', 'Jamie', 'Hodor', 'Drogon', 'Night King'],
             'Daenerys':  ['Ned', 'Arya', 'Sansa', 'Cersei', 'Jamie', 'Hodor', 'Night King'],
             'Cersei':  ['Ned', 'Arya', 'Sansa', 'Jon', 'Daenerys', 'Hodor', 'Drogon', 'Night King'],
             'Jamie':  ['Ned', 'Arya', 'Sansa', 'Jon', 'Daenerys', 'Hodor', 'Drogon', 'Night King'],
             'Hodor':  ['Ned', 'Arya', 'Sansa', 'Jon', 'Daenerys', 'Cersei', 'Jamie', 'Drogon', 'Night King'],
             'Drogon':  ['Ned', 'Arya', 'Sansa', 'Jon', 'Cersei', 'Jamie', 'Hodor', 'Drogon', 'Night King'],
             'Night King':  ['Ned', 'Arya', 'Sansa', 'Jon', 'Daenerys', 'Cersei', 'Jamie', 'Hodor', 'Drogon']}

    def drawNames(self):
        gift_draw = {}   # will store the resulting draw sequence

        stack = []  # keep track of the next vertex to iterate over
        visited = []    # keep track of already visited vertices

        keys = list(self.graph.keys())
        random.shuffle(keys)

        # these two will be used to assign the first drawn person to the last
        first_draw = keys[0]
        last_draw = first_draw

        stack.append(keys[0])

        # traverse the graph until either everyone has been visited or we hit a dead end
        while stack:
            gifter = stack.pop()
            last_draw = gifter
            visited.append(gifter)
            # shuffle vertex neighbours
            random.shuffle(self.graph[gifter])
            # pick a suitable neighbour
            for possible_giftee in self.graph[gifter]:
                if possible_giftee not in visited:
                    gift_draw[gifter] = possible_giftee
                    stack.append(possible_giftee)
                    break

        # "closes the sequence": last drawn participant will buy for the first drawn (if allowed)
        if first_draw in self.graph[last_draw]:
            gift_draw[last_draw] = first_draw

        return gift_draw


def main():
    draw = SecretSantaDraw()
    draw_result =  {}

    # keep drawing until final draw sequence contain all participants
    while not draw_result or len(draw_result.keys()) < len(graph):
        draw_result = draw.drawNames()

    # send results by email
    email = {'Ned': 'ned.stark@gmail.com',
             'Arya': 'arya.stark@gmail.com',
             'Sansa': 'sansa.stark@gmail.com',
             'Jon': 'jon.snow@gmail.com',
             'Danerys': 'khaleesi@yahoo.com',
             'Cersei': 'cersei.lannister@gmail.com',
             'Jamie': 'jamie.lannister@gmail.com',
             'Hodor': 'hodor@gmail.com',
             'Drogon': 'drogon@yahoo.com',
             'Night King': 'nightking@aol.com'}

    port = 465  # For SSL
    password = "<yourpassword>"

    context = ssl._create_unverified_context()

    sender_email = "secretsanta2020throawayaddress@gmail.com"
    with smtplib.SMTP_SSL("smtp.gmail.com", port, context=context) as server:
       server.login(sender_email, password)
       for gifter in draw_result:
           message = "This year you will be buying a gift for....." + draw_result[gifter] + "!"
            server.sendmail(sender_email, email[gifter], str(message))

    print("Secret Santa draw completed!")


if __name__ == "__main__":
    main()
```

Here is an example output for the draw sequence when running this script:

```
{'Drogon': 'Night King', 'Night King': 'Jon', 'Jon': 'Jamie', 'Jamie': 'Sansa', 'Sansa': 'Cersei', 'Cersei': 'Hodor', 'Hodor': 'Arya', 'Arya': 'Daenerys', 'Daenerys': 'Ned', 'Ned': 'Drogon'}
```

A few comment about the code: 
- It is not guaranteed that after the while loop, the resulting sequence will contain all the participants. It could be that the loop terminated because of a dead end, or that we only traversed a subset of the graph that formed a cycle. That's why we need to check that the resulting sequence contain all the participants before sending the results. Note that if we provide an input with no solution, this program will not terminate as the `draw_names` function will be called indefinitely.
- The `stack` array doesn't have to be a stack nor an array. Its sole purpose is to keep track of the next vertex to "traverse", so it could be a simple variable. For some reason it made sense in my head to think of it as a stack, maybe because this traversal is vaguely similar to a DFS.
- For sending emails, I casually copy pasted the solution proposed in [realpython](https://realpython.com/python-send-email/), with a throwaway gmail account created for the occasion. 

Here's a piece of code that I'm hoping to reuse for the rest of my life. If you also do secret santas, feel free to use it too!
