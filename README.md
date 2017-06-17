# Paxboards

A simple, extensible, threaded bulletin board system for Evennia, with both on-game and web-based interfaces.  The on-game interface is modeled loosely after Firan's fork of Myrddin's BBoard system, or after Arx's bboard system.  The web interface is modeled loosely after phpBB type web-forums.

Paxboards was originally written by Packetdancer, a.k.a. "Pax", for use on Aurora Chain.

## Installation

### Basic Requirements

Simply grab this repository as a subdirectory of your Evennia game directory, and then add the following two lines to your `server/conf/settings.py` file:

```
INSTALLED_APPS += ('paxboards',)
TYPECLASS_PATHS += ['paxboards']
```

Then go to `web/urls.py` and add:

```
custom_patterns = [
    url(r'^boards/', include('paxboards.urls', namespace='board', app_name='paxboards')),
]
```

If you already have custom patterns, just add the url record to your existing list.

Lastly, you will need to copy `paxboards.css` from the `templates` directory of the paxboards installation to your `web/static/website/css` directory.

When all of this is done, run `evennia makemigrations paxboards` and `evennia migrate`, then execute `@reload` on your game, and you should be good to go.  

You can use the `bbadmin` command on your game to create a test board.

### Updating Templates

If you want to link the boards from anywhere on your website, simply use `{% url 'paxboards:boardlist' %}` in any template file to automatically generate the appropriate URL for your site installation.

### Adding to Django Admin Console

If you want to administer bboards from the Django web admin console, you'll also need to modify your `evennia_admin.html` in order to add the following to the list of administrative sections:

```
    <h2><a href="{% url "admin:paxboards_boarddb_changelist" %}">Boards</a></h2>
    Boards are used for static player communications.
```

## Components

### DefaultBoard

DefaultBoard is a new Evennia base typeclass, which encapsulates all the board functionality.  It shows up at a database level and is manageable as with any other Evennia typeclass.

It also exposes a custom Django manager/queryset handler, allowing you to easily query which bboards are visible to a specific player from the class itself.

It supports full lock handlers, with the following access keys:

* `read`: can read the board -- _this is necessary to see the board in your list_
* `post`: can post to the board
* `edit`: can edit posts by people other than yourself
* `delete`: can delete posts by people other than yourself
* `pin`: can pin or unpin posts from the board

### Post

Post is a simple object class which encapsulates a given post.  Boards can be asked for their posts, and when you provide a calling player, it will annotate each Post with an 'unread' property as to whether or not that property has been read.

It supports some simple tools to check whether or not a player has access to perform a given operation.

## TODO

* As this was my first major Evennia code and I was just off in my own corner with it, there's probably places I could've done things more 'properly' by an Evennia standard (instead of a Django standard with Evennia-ish bits thrown in):
	* For instance, DefaultBoard doesn't really need to be set up as a typeclass, or if it is, it should use normal dbrefs.
	* If they used normal dbrefs, then tags/attributes could be used instead of the `db_*` fields that are presently used to configure boards.
	* The entire thing probably didn't need to be made a Django installable application; it may not fit well into normal Evennia contrib as a result.
	* The DefaultBoard/Post APIs could be cleaned up considerably; we probably shouldn't be dipping into the models' `db_*` fields for basic access outside of the classes themselves.  I started a bit of this.
* The CSS/HTML styling for the actual threads could definitely be better.
* The web-side needs proper web-forum-style paging, so you can get smaller 'chunks' of a board at once.
* The helpfile for bboard could be a lot better in general.
