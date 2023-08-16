---
layout: post
title:  "A trip around the world from the comfort of your Spotify account"
date:   2023-08-02
categories: projects show-and-tell
---

# A trip around the world from the comfort of your Spotify account

I like music. In particular, I like discovering new music that's different to the music I've already heard. The desire to find that different music has led to my Spotify playlist containing music spanning from [Clown Core](https://open.spotify.com/album/4arqmekzS1Qnzu74ZTl7uV?si=GQ1k849GRRyNrXQDjsSjFw) to [Pokemon Pickup Lines](https://open.spotify.com/track/4QeYZLyC3u4Fv4XNVT6LAL?si=8edd099d3f254104) to [Mongolian Folk Metal](https://open.spotify.com/album/74cgx2ViWpE9Lv1MTGOqrl?si=YvSFG2DlQ5qR3AL50rPiZw).

While it can be fun seeking out new interesting music, I've come to enjoy the tastes of new music I get on Mondays and Friday's in Spotify's auto-generated _Discover Weekly_ and _Release Radar_ playlists. What I wanted was another weekly playlist that generated much more varied suggestions than those two could provide.

While there's no short supply of styles of music performed by English-speaking musicians, I felt that musicians writing music in other countries and languages were at least _more_ likely to have styles that were different to the music I'd listened to before. As such, I set out to set up a system to deliver me a new playlist that somehow pulled song recommendations from several different counties. 

## How do?

Aside from the playlists automatically generated in members' accounts, Spotify also generates various shared playlists including ones that aggregate the current 50 most popular songs (by whatever metrics Spotify uses to evaluate their popularity). Those _Top 50_ playlists are generated in many (though not all) of the regions in which Spotify operates and are a great source from which to pull recommendations of new music. 

Only having the 50 most popular songs from each country as candidates for my playlist does mean that I'm not going to be discovering any niche artists. There is, however, a high chance that I, living in South Africa, have never heard of the majority of artists on the Top 50 playlist in Thailand; they're new to me, so for now, I'm not too hung up on the fact that I'm likely hearing the most commercial music that each country has to offer.

So, the plan, at a high level was:
1. Get a list of every country in which Spotify operates
2. Use that list to find every existing regional Top 50 playlist
3. Every Wednesday morning, pull 40 random songs from those playlists and use them to create a Global Playlist

## Making it happen

You can find the source code for the project [on GitHub](https://github.com/BrydonLeonard/GlobalPlaylist). It's not a hugely complex system (it's really just the steps I listed in the previous section), so here I'll just run through some of the more interesting bits of functionality. 

### Finding markets

Spotify is available in many countries around the world (which they call "markets") and makes the list available through [a convenient markets API](https://developer.spotify.com/documentation/web-api). The list of markets is a list of [ISO3166 country codes](https://en.wikipedia.org/wiki/ISO_3166). 

I grabbed [a list of country codes from](https://en.wikipedia.org/wiki/List_of_ISO_3166_country_codes) and manually turned them into JSON for use at runtime. That way, I could easily turn `ZA` into a search for `Top 50 South Africa`.

### Cloudy with a chance of meatballs

I wanted the script to run automatically every week, but also don't currently have a home server (or always-on PC) from which to run in, so I ran it in the cloud! There's nothing groundbreaking (or even particularly complex) in the way that it works. Once a week on a Wednesdays, a CloudWatch event triggers a Lambda where all of the business logic runs. Anything that I need to store between runs goes in DDB.

![architecture](/assets/images/global-playlist/architecture.PNG)

With the architecture being as basic as it is, I didn't bother with CloudFormation and instead clicked through the console to create the necessary resources. I then implemented [a basic caching client](https://github.com/BrydonLeonard/GlobalPlaylist/blob/mainline/global_playlist/cache.py) to encapsulate the DDB interactions.

### Constraints

After the first playlist was generated, I realised that I wanted to add some additional constraints:
- Some artists are a little _too_ popular and dominate the Top 50 playlists of many countries simultaneously. As a result, instead of being an interesting selection of music from around the world, the playlist ended up being mostly Bad Bunny and Harry Styles the first time it was generated. To fix that, I added [a constraint that prevented an artist from having more than one song in the playlist](https://github.com/BrydonLeonard/GlobalPlaylist/blob/mainline/global_playlist/song_provider.py#L23).
    - If song has multiple listed artists, none of them can have any other songs in that week's playlist
- Because many songs at the top end of the Top 50 playlists end up hanging around for several weeks, I started to see those showing up in the playlists for multiple weeks in a row. [I added a DDB table to keep track of the songs that had already been in the playlist](https://github.com/BrydonLeonard/GlobalPlaylist/blob/mainline/global_playlist/song_provider.py#L25-L29) to ensure that they only ever appeared once.
- Not all markets in which Spotify operates have their own Top 50 playlists. Many of those that don't, however, have similar "Viral 50" or "Hot" playlists, so I [expanded the search to include those where necessary](https://github.com/BrydonLeonard/GlobalPlaylist/blob/mainline/global_playlist/song_provider.py#L109).


## The results

Here it is! 1x weekly global playlist:

<iframe style="border-radius:12px" src="https://open.spotify.com/embed/playlist/5kJaBnbdb5vQDZ0UfhsAgy?utm_source=generator&theme=0" width="100%" height="352" frameBorder="0" allowfullscreen="" allow="autoplay; clipboard-write; encrypted-media; fullscreen; picture-in-picture" loading="lazy"></iframe>


### Future development
There are several developer experience improvements around updating the Lambda, getting hold of Spotify API tokens, and invalidating the song cache that I could look at. Given that the workflow is pretty much fire-and-forget (I've touched it once since I set it running), there's no need to make those one-time processes easier yet.

With a couple more hours of effort, I could also have the script generate playlists in other accounts to let users have their own global playlists, but they'd all pull from the same set of "Top 50" playlists and would all end up being pretty similar. An interesting extension of this idea would be to allow users to specify the playlists from which they'd like to pull every week. That'd allow them to curate the sources of music to some extent while still getting a pleasant surprise when the playlist is re-generated.
