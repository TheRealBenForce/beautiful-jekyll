---
layout: post
title: A Look At Facebook's Shadow Profiles
tags: [security, social networking, facebook]
---

I'm hot off the heals of passing the CISSP exam and felt inclined to post on a security related topic. Perhaps a bit dystopian in nature but given light of recent revelations regarding data collection from Cambridge Analytical by means of Facebook, I thought I'd write a bit about a theory I developed a few years back regarding collection of data for non-users of Facebook, AKA "shadow profiles". This theory came to me after reading about [facial recognition at massive scale](https://www.huffingtonpost.com/2014/03/18/facebook-deepface-facial-recognition_n_4985925.html) for the first time which happened to be on the Facebook platform.

[Kashmir Hill from Gizmodo writes a good article](https://gizmodo.com/how-facebook-figures-out-everyone-youve-ever-met-1819822691) about the People You May Know feature in Facebook for contact mapping. But this is mostly in regards to building your social network and doesn't mention the advertising element, which is what I'd like to postulate today. We don't exactly know what Facebook is doing with non-user data. Facebook has been vague and hush about it. Humor me for today and suspend any disbelief. It is all entirely plausible

I believe Facebook has a mission to create a profile for every living person on the planet whether they have signed up for site or not. The reason to create a "shadow" profile on non subscribed users is so that when they do formally sign up, highly targeted adds can be directed to them immediately, even before the new user formally provides much information.

This begins with facial recognition software. What may be happening behind the scenes is that Facebook attempts to ID every face for every photo uploaded to the site, even if the face was just a subject in the background. As a repeat unknown face is identified over and over again, the platform begins building a shadow profile on the unidentified person. After a high enough confidence threshold is reached, the platform will determine the shadow profile is a legitimate new user on the platform and use what the platform has learned about the shadow profile to more accurately target advertisements to the user. This also improves ad targeting to other users by removing the identified shadow profile from the pool of shadow profile candidates to other new users. This is especially useful for new users or users who have not explicitly provided much information in their profile.

Consider this example:

* An untagged face that can not be associated to a user account is identified by the facial recognition software. The software determines this face does not exist on the social media platform and is assigned a unique identifier, we will call UNKNOWN1.
* Some attributes can be extrapolated immediately from the face:
  * Age
  * Sex
  * Geotagging in the image of the picture's location
* Each time a new photo with unknown faces are uploaded to the platform, they are compared to all the unknown faces in the shadow profile database and a high confidence threshold matches some of these faces to UNKNOWN1.
* As the UNKNOWN1 appears in more and more photos, common themes are identified by the platform. For example
  * The face appears most commonly by a group of users who live in zip codes within 25 miles of each other.
  * The face doesn't typically appear Monday to Friday between 11PM and 5PM. UNKNOWN1 is sleeping and working during this time.
  * Outside working and sleeping hours, UNKNOWN1 often appears in photos with a geotag pointing to a local fitness club.
  * UNKNOWN1 appeared in photos that were taken at a local 5K event.
  * UNKNOWN1 appeared in photos that were taken at the local farmers market.
  * Throughout the winter UNKNOWN1 appeared in the background of a few photos taken at a ski resort.
  * UNKNOWN1 appears most commonly in 4 user's photos. The platform determines UNKNOWN1 is likely a primary subject in these photos rather than a person in the background and UNKNOWN1 is most likely to have a profile similar to theirs as opposed to all the other photos UNKNOWN1 has appeared in.
    * These 4 users all graduated as undergraduates from Penn State University in 2010.
    * These 4 users also all added several romantic comedies staring Jason Segel to their lists of favorite movies.

Facebook now has a robust profile built around UNKNOWN1. With a pretty good confidence level, the platform knows UNKNOWN1's age, sex, neighborhood, that the person likes to eat healthy, is into fitness, likes romantic comedies with Jason Segel, graduated from Penn State University in 2010, and likes to or is learning to ski or snowboard.

Now, a new user signs up to the site we will call NEWUSER1. Being that Facebook is essentially an advertising platform, it needs to produce some ads even for new users that have not provided much information. This is actually critical for a few reasons:

* The users are most likely to notice the ads at this time due to the lack of other distractions.
* It is difficult to convert established users to ad clickers.

Getting new users into this habit early is essential. So how does Facebook then pick the ads to send NEWUSER1 since it knows very little about this person? Here is how.

NEWUSER1 was invited to the site by a person that uploaded a picture that contained UNKNOWN1. NEWUSER1 then sends an friend requests to 2 users that also uploaded photos that UNKNOWN1 was tagged in. Despite NEWUSER1 never uploading a photo or providing their interests or any information besides what is minimally required and sending two friend invites, the platform must decide what ads to send. It does so by selecting the shadow profile that NEWUSER1 most likely is and sends NEWUSER1 the ads that a profile like the shadow profile might receive. The platform concludes NEWUSER1 is most likely UNKNOWN1. NEWUSER1 receives advertisements for ski supplies, fitness gear, a new romcom movie coming out with Jason Segel, and Phoenix University graduate school. As NEWUSER1 begins to add more and more friends and interests, Facebook continues to grow its confidence that NEWUSER1 is UNKNOWN1. Eventually NEWUSER1 uploads a photo. The confidence threshold is high enough for Facebook to merge UNKNOWN1 to NEWUSER1.

Taking it one step further, this whole time there was also NEWUSER2. NEWUSER2 also had UNKNOWN1 as its highest confidence shadow profile. Both NEWUSER1 and NEWUSER2 were essentially receiving the same advertisements. But not anymore, because NEWUSER1 is UNKNOWN1, NEWUSER2 can't be UNKNOWN1. SO NEWUSER2 starts receiving advertisements that the next highest confidence shadow profile would receive, UNKNOWN2. UNKNOWN1 and UNKNOWN2 are so similar its scary. The only difference is that UNKNOWN2 prefers any movie with James Franco. This happens at lightspeed, every second, of every day, for every picture.
