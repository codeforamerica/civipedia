Citihow is an online community of practice for anyone who wants to make their city better. 

CfA’s OSQA aims to support two kinds of leaders -- beginners and
experts. For the experts -- anyone from city staff to local neighborhood
mavens -- we hope to create an easy avenue to share their knowledge of
special interest and get recognized for it. For beginners -- anyone
newly inspired to get involved -- we hope to create a rich library of
local expertise to draw from, so that starting from scratch isn’t so
daunting.

There is a wide gap between intention and action. We believe people want
to have pride in their communities and will volunteer their time to
improve their neighborhoods, be that cleaning up a street, hosting a
block party or starting a community garden. But taking the first step
can be the hardest, and the sooner initial questions are answered, the
more likely positive change and civic leadership will happen. 

Deploying Citihow to DotCloud

Sign up for a free dotcloud account
    https://www.dotcloud.com/accounts/register/

Mac OS X
Install the dotcloud command line tools
Open Terminal and run the following command
    sudo easy_install pip && sudo pip install dotcloud

When the installation is finished, run “dotcloud” for the first time and enter your API key.
Your API key is displayed in your Settings page
https://www.dotcloud.com/accounts/settings

    $ dotcloud
    Enter your api key (You can find it at http://www.dotcloud.com/account/settings): 

Clone a copy of the citihow repository to a new directory
    git clone https://github.com/codeforamerica/civipedia citihow

Create a dotcloud namespace for your application (you can use any name,
but for this example we'll use citihow)
    dotcloud create citihow

Change to the citihow directory, which should have a dotcloud.yml file
in it
    cd citihow

*** This gets a little more complicated, we're working on simplfying
these steps but until then follow along closely ***

Push the code to the dotcloud applicaiton
    dotcloud push citihow

You'll need the last line that's displayed from this command for a later
step. It should look like this:
dev: http://4fad5132.dotcloud.com/

Create a new settings file for citihow
    cp dev/catalyze/settings_local.py.dist dev/catalyze/settings_local.py

Now we need to get the connection details for the database from the
dotcloud application. Dotcloud creates a file on the python server with
all of the details and allows us to access it over ssh.
    dotcloud ssh citihow.dev

View the database settings by typing. You'll want to copy this
information because you'll need it for the next couple of steps.
    cat environment.json

Close the dotcloud connection by typing:
    exit

Edit the settings file you created:
    nano dev/catalyze/settings_local.py

Update these lines
    27 DATABASE_NAME = ''             # Or path to database file if using sqlite3.
    28 DATABASE_USER = ''               # Not used with sqlite3.
    29 DATABASE_PASSWORD = ''               # Not used with sqlite3.
    30 DATABASE_ENGINE = ''  #mysql, etc
    31 DATABASE_HOST = ''
    32 DATABASE_PORT = ''

Using these values from the environment.json file (this is an example,
make sure to use the values from the earlier step)
    "DOTCLOUD_DB_MYSQL_LOGIN": "root", 
    "DOTCLOUD_DB_MYSQL_PASSWORD": "long_random_string", 
    "DOTCLOUD_PROJECT": "citihow", 
    "DOTCLOUD_SERVICE_NAME": "dev", 
    "DOTCLOUD_DB_MYSQL_PORT": "four_digit_number", 
    "DOTCLOUD_DB_MYSQL_HOST": "random_string.dotcloud.com", 
    "DOTCLOUD_SERVICE_ID": "0"

So that they look like this
    27 DATABASE_NAME = 'citihow_dev'             # Or path to database file if using sqlite3.
    28 DATABASE_USER = 'root'               # Not used with sqlite3.
    29 DATABASE_PASSWORD = 'long_random_string'               # Not used with sqlite3.
    30 DATABASE_ENGINE = 'mysql'  #mysql, etc
    31 DATABASE_HOST = 'random_string.dotcloud.com'
    32 DATABASE_PORT = 'four_digit_number'

Then change:
    41 APP_URL = 'http://'
To add the line from the earlier step so it looks like
    APP_URL = 'http://randomchars.dotcloud.com/'

Exit nano by typing ctrl-X and hit Y then enter to save and overwrite
the file.

Push the code to Dotcloud again to update the configuration
    dotcloud push citihow

Log into the dotcloud mysql service
    dotcloud ssh citihow.db

Run mysql as the root user and paste in the DOTCLOUD_DB_MYSQL_PASSWORD
when prompted
    mysql -u root -p

Create the database for citihow
    CREATE DATABASE citihow_dev DEFAULT CHARACTER SET UTF8 COLLATE utf8_general_ci;

Exit mysql
    exit;

Close the remote connection to the dotcloud mysql service
    exit

Use the dotcloud CLI to run a remote command on the python service
    dotcloud run citihow.dev -- python /home/dotcloud/current/catalyze/manage.py syncdb --all
    dotcloud run citihow.dev -- python /home/dotcloud/current/catalyze/manage.py migrate forum --fake

Open a web browser and go to the APP_URL that was listed above, you
should see the citihow home page
http://randomchars.dotcloud.com/


[![Code for America Tracker](http://stats.codeforamerica.org/codeforamerica/civipedia.png)](http://stats.codeforamerica.org/projects/civipedia)
