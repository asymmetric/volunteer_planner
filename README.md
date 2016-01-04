# volunteer-planner.org

Volunteer Planner is a platform to schedule shifts of volunteers. Volunteers register at the platform and choose shifts.
 The admin of the website can easily add new organizations, places and shifts. The software has a location based 
 hierarchy (country / region / area / city) and has a hierarchy of organizations (organizations, facilities, tasks and
  workplaces) - it can be used for a variety of purposes.

## Status
The project is currently running at https://volunteer-planner.org/.

## Work in progress
There are some feature requests to be implemented in the future.
The software currently needs a centralized administration of the shifts, but it is one of the main goals of the current 
development to empower organizations to schedule shifts for their facilities on their own.

If you are interested to join the development team, just make pull requests or come to a meeting in Berlin/Germany: 
http://www.meetup.com/de/coders4help/

## System context
**User**: The volunteers and administrators just need a (modern) web browser to use the volunteer-planner application.

**Developer**: Developers need a python development environment (see project setup) and specific versions of external 
libraries (see /requirements directory, t). Development can be done with a sqlite databases, there is no need to run 
and configure postgres or mysql.

**Server**: For production use you need a Python ready web server, for example uWSGI as web server for the Python WSGI 
with nginx as proxy server visible to the end user (volunteers and administrators). You also need a MySQL or PostgreSQL 
database.


## Project setup for development

### 0. Install docker

Follow [the instructions](https://docs.docker.com/engine/installation/) relevant to your operating system.

### 1. Fork us on GitHub

Please fork us on GitHub and clone your fork

    git clone https://github.com/YOUR_GITHUB_ACCOUNT/volunteer_planner.git

### 1.1 Creating Pull Requests

Please do Pull Requests against the [`develop` branch](https://github.com/volunteer-planner/volunteer_planner/tree/develop).

If you have questions concerning our workflow please read the 
[Development Rules wiki page](https://github.com/volunteer-planner/volunteer_planner/wiki/DevelopmentRules).

### 2. Prepare the DB

By default, the application will be using MySQL. If you prefer PostgreSQL, edit
the `Dockerfile` and `docker-compose.yml` files, uncommenting the PostgreSQL
related parts, and commenting out the MySQL related ones.

#### 2.1 Initialize the DB container

Run `docker-compose up db`, and wait for the initialization to be over. You can now kill the container with `CTRL-c`.

#### 2.2 Run migrate management command to setup non-existing tables 

    $ docker-compose run web python manage.py migrate

#### 2.3 Add a superuser

    $ docker-compose run web python manage.py createsuperuser

You will be asked for username, email and password (twice). Remember that
username and password.

### 3. Try running the server

    $ docker-compose up

Try opening http://localhost:8000/ in your browser.

### 5. Adding content

To add new organizations and shifts, you have to access the backend at
`http://localhost:8000/admin`. If prompted, login with the username/password of
the superuser you created earlier (in case you don't see an error page here).

    http://localhost:8000/admin

## The Project

### Create Dummy Data

Run management command

    $ docker-compose run web python manage.py create_dummy_data 5 --flush True

with activated virtualenv to get 5 days of dummy data and delete tables in advance.

The number (5 in the above example) creates 5 days dummy data count from today.
If you just use `./manage.py create_dummy_data 5` without `--flush` it is NOT deleting data before putting new data in.

### Running Tests

Feature pull requests should be accompanied by appropriate tests. We have unit and integration tests that
are run with `py.test`, and functional/behave tests that are run with `selenium`.

To run unit tests, run the following command (with your virtual env activated, see 3.)

    $ docker-compose run web py.test -v [/path/to/volunteer_planner.git/]

If you want to generate a coverage report as well, run

    $ docker-compose run web py.test --cov=. --cov-report html --cov-report term-missing --no-cov-on-fail -v

This generates a nice HTML coverage page, to poke around which can be found at `/path/to/volunteer_planner.git/htmlcov/index.html`. 

*Note*: The directory `htmlcov` is git-ignored.

To run selenium tests, run

    $ behave tests/_behave

### Translations

We use [Transifex (tx)](https://www.transifex.com/coders4help/volunteer-planner/) for managing translations.

#### General notes

* Please read 
    * [Django 1.8: Internationalization and localization](https://docs.djangoproject.com/en/1.8/topics/i18n/)
    * [Django 1.8: Translations](https://docs.djangoproject.com/en/1.8/topics/i18n/translation/)
* Please avoid internationalized strings / messages containing HTML markup. This makes the site layout depending on the translators and them getting the markup right; it's error prone and hardly maintainable when the page's layout changes.
* use `trimmed` option in [blocktrans](https://docs.djangoproject.com/en/1.8/topics/i18n/translation/#std:templatetag-blocktrans) template tags, if indention is not intended.
* Please provide [contextual markers](https://docs.djangoproject.com/en/1.8/topics/i18n/translation/#contextual-markers) on strings to help translators understanding the usage of the strings better. The shorter an internationalized string is, the more abigious it will be and the more important an contextual hint will be.

#### Workflow

1. Code your stuff using the `ugettext_lazy as _` etc. methods to mark internationalized strings
2. Update the po files `./manage.py makemessages --no-wrap --no-obsolete -l en`
   The options are intended to make the output more git-friendly.
3. Push the updated translations to git. **Do not intend to translate in the local .po files, any changes here will be overwritten when translations are pulled from [tx](https://www.transifex.com/coders4help/volunteer-planner/).**
4. Transifex will automatically update the source strings via github once a day and make them available for translation. 
4.1. If necessary, translation managers (meaning VP's Transifex project admins) can update the source language manually using the tx client command `tx push -s django`. 
5. Translators will then translate on [VP's Transifex project](https://www.transifex.com/coders4help/volunteer-planner/)
6. When new translations are available on Transifex `tx pull` will update the local .po files with translations from TX
7. `./manage.py makemessages --no-wrap --no-obsolete` will reformat po files in a more readable single-line message string format
8. `./manage.py compilemessages`
9. Test if it looks good and works as intended
10. Commit and push the updated translations to git

Your local installation should be translated then. The .mo file created by compilemessages is gitignored,
you'll need to (re-)generate it locally every time the .po file changes.

#### How to use the Transifex client

You first need to make sure that the transiflex client is installed (should be in the requirements/dev.txt file).

```
pip install transifex-client
```

* For further installation infos and setup read [Transifex: Client setup](http://docs.transifex.com/client/config/)
* Then, sign up at https://www.transifex.com, search for the project volunteer-planner.org, and join the respective team.
* If you used an Oauth-ish method (Google, Facebook, etc.) to sign up for Transifex, you might need to set a password in your [Transifex profile](https://www.transifex.com/user/settings/password/) before you can use the client.
* Edit your personal transifex configuration file that is stored in your home directory at ~/.transifexrc
```
[https://www.transifex.com]
username = YOUR_TRANSIFEX_USERNAME
token =
password = YOUR_TRANSIFEX_PASSWORD
hostname = https://www.transifex.com
```
Make sure not to share this file with anyone, as it contains your credentials! For more information on configuration, see http://docs.transifex.com/client/config/
