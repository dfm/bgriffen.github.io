---
layout: post
title: "Making Spotify Playlists In Python Based On Upcoming Local Bands"
description: "Want to see some live music, but not sure if you'll like the band? Try this out."
tags: [python, spotify, music]
---

If you know a band it is easy to Google or search for them on Spotify to hear their music. This is great if you already know the music you are looking for. What about for bands you haven't heard of who happen to be playing a live show in the coming months in your area? The thought of compiling this list manually makes me shudder and this sort of thing lends itself very well to automation via a program.

I decided create just that; a short program which finds bands with upcoming shows in your area (Cambridge/Boston, MA in my case). My data source: [The Bowery Boston](http://www.boweryboston.com/see-all-shows/).

[![spotify-playlists](/assets/spotifylocalbands/bowery_boston.png)](/assets/spotifylocalbands/bowery_boston.png)

Using this list I then construct a Spotify playlist containing the top few tracks of each of the bands about to have a show. I run this once a month and play it periodically to see if there is any new music which catches my ear then if I'm interested enough, I'll go ahead and buy a ticket.

My only point of reference was [this nice post](https://mborgerson.com/creating-a-playlist-in-spotify-using-python/) outlining the frame work by Matt Borgerson. His [original code](https://github.com/mborgerson/spotify-playlist-from-csv) convereted a csv file into a playlist - which is a great tool if you already know what bands *and* tracks you want to listen to. I had to design something slightly different.

For this code to work you require Python, libspotify, the pyspotify bindings and critically a Premium Spotify account ($9.99/month, sorry). Code can be found at [this Github repository](https://github.com/bgriffen/spotifylocalbands).

{% highlight Python %}
import requests
import bs4
import spotify
import sys

"""
Contact: Brendan Griffen brendan.f.griffen@gmail.com @brendangriffen

Code will scrape bands playing in next few months in Boston area.

Requirements:

libspotify:     https://developer.spotify.com/technologies/libspotify/
pyspotify:      https://github.com/mopidy/pyspotify
developer key:  https://devaccount.spotify.com/my-account/keys/

Enjoy the tunes!

"""

session = spotify.Session()
session.login('username', 'password')

url_name = "http://www.boweryboston.com/see-all-shows/"

response = requests.get(url_name)
soup = bs4.BeautifulSoup(response.text)

# scrape the relevant information and put into a list
bands_split = [elm.a.text.split(",") for elm in soup.find_all('h1',class_='headliners summary')]

# don't forget the support acts!
supports = [elm.a.text for elm in soup.find_all('h2',class_='supports')]

bands = sum(bands_split,[])

all_bands = list(set(bands+supports))

print "Upcoming bands playing around Boston."

for band in all_bands:
    print band

# try a few times to connect
for i in xrange(0,5):
    session.process_events()

print
print "Adding TOP 3 songs of each band to a Spotify playlist..."
all_tracks = []
for band in all_bands:
    search = session.search(str(band))
    search.load()
    if len(search.tracks) > 0:
        # take the top 5 tracks
        for i,track in enumerate(search.tracks):
            if i <= 2: all_tracks.append(track)

print "Adding %i band, totalling %i tracks!" % (len(all_bands),len(all_tracks))
session.playlist_container.add_new_playlist("Upcoming LIVE Boston Music")
playlist = session.playlist_container[-1]

print "Adding tracks to:",playlist.name
playlist.add_tracks(all_tracks)
playlist.load()

print
print "Check your new playlist soon!"

# check the playlist now exists!
for playlisti in session.playlist_container:
    print playlisti.name

session.logout()

print
print "Check your new playlist soon!"

{% endhighlight %}

The terminal output will be something like the following:

{% highlight bash %}
Upcoming bands playing around Boston....
Wild Child
Future Islands
Caroline Smith
Rohan Padhye
Sylvan Esso
The Green
Sidewalk Driver
The Fagettes
Abhishek Shah
Front Porch Step
Cropduster
Preservation Hall Jazz Band
Sorority Noise
The Beautiful Ones
...
Adding TOP 3 songs of each band to a Spotify playlist...
Adding 222 band, totalling 541 tracks!
Adding tracks to playlist: Upcoming LIVE Boston Music
{% endhighlight %}

*Small Caveat: sometimes the program will scrape the wrong band - please forgive me.*


It may take quite some time to show up in your playlist. Here is an example of what one of these playlists might look like inside Spotify (a restart sometimes helps):

[![spotify-playlists](/assets/spotifylocalbands/spotify_playlist.png)](/assets/spotifylocalbands/spotify_playlist.png)

I made [this playlist public](http://open.spotify.com/user/1254170771/playlist/5QKiOM9egThI6u6oXgkTNh) for those that aren't too comfortable programming so feel free to follow it if you are in the Boston area (this will work if you have a free account). I'll use my Premium account update it periodically =)

Just enjoy the new music, soon to be played live somewhere near you.
