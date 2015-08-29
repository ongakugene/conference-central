App Engine application for the Udacity training course.

## Products
- [App Engine][1]

## Language
- [Python][2]

## APIs
- [Google Cloud Endpoints][3]

## Setup Instructions
1. Update the value of `application` in `app.yaml` to the app ID you
   have registered in the App Engine admin console and would like to use to host
   your instance of this sample.
1. Update the values at the top of `settings.py` to
   reflect the respective client IDs you have registered in the
   [Developer Console][4].
1. Update the value of CLIENT_ID in `static/js/app.js` to the Web client ID
1. (Optional) Mark the configuration files as unchanged as follows:
   `$ git update-index --assume-unchanged app.yaml settings.py static/js/app.js`
1. Run the app with the devserver using `dev_appserver.py DIR`, and ensure it's running by visiting your local server's address (by default [localhost:8080][5].)
1. (Optional) Generate your client library(ies) with [the endpoints tool][6].
1. Deploy your application.


## Solutions for the tasks:

App ID:  hamd-o-sanaa

Note: 
For testing the following existing keys can be used:
```
Conference Key:	ag5zfmhhbWQtby1zYW5hYXI4CxIHUHJvZmlsZSIbbmlraGlsLmtodWxsYXI3ODZAZ21haWwuY29tDAsSCkNvbmZlcmVuY2UYAQw
Session 1 Key [Speaker - Nik]: ag5zfmhhbWQtby1zYW5hYXJHCxIHUHJvZmlsZSIbbmlraGlsLmtodWxsYXI3ODZAZ21haWwuY29tDAsSCkNvbmZlcmVuY2UYAQwLEgdTZXNzaW9uGKGcAQw
Session 2 Key [Speaker - Ali]: ag5zfmhhbWQtby1zYW5hYXJHCxIHUHJvZmlsZSIbbmlraGlsLmtodWxsYXI3ODZAZ21haWwuY29tDAsSCkNvbmZlcmVuY2UYAQwLEgdTZXNzaW9uGLHqAQw
```

## 1. Sessions can be added to a conference
About design decisions:
For session implementation, I have developed it as child of the conference. This is because as logically session will be there only in a conference and thus conference (parent) needs to exist first before a session can be vreated under it. A Session can be created with only a valid Conference Key and a name, other fields are provided with sensible defaults. Speaker is just a string value representing the name and thus is indexed and used for searching by default. I have created a SessionType class to hold enumeration values for typeOfSession. The design decision is similar to that for T-Shirt sizes and ensures that arbitrary values never get entered into this field or in other words, we only get from amongst a standard set of values for this field. I have chosen TimeProperty for the startTime values.

Name of session is a required field while since there can be multiple highlights, it has repeated set to true.

## 2. Sessions can be added by a user to their wishlist

## 3. Two Additional queries and Query-Problem:
```
# 3a. Get all the sessions held in London
conferences = Conference.query(Conference.city=="London")
sessions = []
for conf in conferences:
	sessions += Session.query(ancestor=conf.key)
```	
```
# 3b. Get conferences with less than 20 people attending
conferences = Conference.query(Conference.maxAttendees<=20)
```

# 3c. Query problem
The query has two conditions for inequality. This leads to the following error:
```
BadRequestError: Only one inequality filter per query is supported.
```
This can be solved by using "IN" instead of != for one of them as follows: 
```
time_last = datetime.strptime("19:00", "%H:%M").time()
# Getting all types other than WORKSHOP
allowed_types = [key.name for key in SessionType if key != SessionType.WORKSHOP]
for session in Session.query(ndb.AND(Session.typeOfSession.IN(allowed_types), 
                                     Session.startTime < time_last)):
    print '%s of type %s at %s' % (session.name, session.typeOfSession, 
                                session.startTime.strftime("%H:%M"))
```

## 4. Memcached Task
When a new session is added to a conference, the speaker is checked. If there is more than one session by this speaker at this conference, a new Memcache entry is also added that features the speaker and session names.


[1]: https://developers.google.com/appengine
[2]: http://python.org
[3]: https://developers.google.com/appengine/docs/python/endpoints/
[4]: https://console.developers.google.com/
[5]: https://localhost:8080/
[6]: https://developers.google.com/appengine/docs/python/endpoints/endpoints_tool