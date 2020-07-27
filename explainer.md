# Media Feeds Explainer

Author: beccahughes@chromium.org

Last modified: 2020-02-06

## Objectives

User agents are adding much more functionality around media on the web. For example, Chromium browsers added [media controls](https://blog.google/products/chrome/manage-audio-and-video-in-chrome/) to the browser UI to control playback on the web.

To improve the functionality of such features we want to be able to add support for sites to recommend media content to a user that might be completely new or they might be something the user has started watching. This allows us to deliver a much better experience to users.

Media Feeds provides a way for user agents to discover such feeds and provides a template for how sites should format their data.

### Goals

-   A site tells the user agent it has a feed
    
-   A site should be able to recommend relevant content for the user. A site should be able to recommend media content to a user that is relevant to them. If the user is not logged in this can be generic.
    
-   A site should be able to recommend content for a user to continue watching. A site should be able to recommend content that a user has started watching and provide that in a way so the user agent can identify to the user they have already started watching this content.
    
-   A site should be able to recommend content for a user to play next. In cases where the media is a TV series the site should be able to recommend the user to start watching the next episode and/or season.
    

### Non-goals

-   Media feeds are currently limited to video content. Audio-only content such as music or podcast are out of scope for now.
    
-   [MediaRSS/RSS](http://www.rssboard.org/media-rss#media-content) is an alternative, however it does not give us enough detail about whether the content is a TV show, movie as well as the structure around a TV series and seasons.

-   Media Feeds is intended to provide data out-of-band while the page is closed. Providing metadata about the currently playing item or while the tab is open is not a goal for Media Feeds.


## Design

The website should advertise a media feed to the user agent by adding a `link` element to the head of the document. The `rel` attribute should be set to `media-feed`. The `href` attribute should be on the same origin as the document URL. A user agent will automatically discover the media feed and store it to be fetched later.

```html
<head>
  <link rel="media-feed" href="https://www.example.com/media-feed">
</head>
```

Media Feeds are feeds of JSON data and contain data using the existing [schema.org](https://schema.org) standard. The user agent will fetch the media feed using a `GET` request with the appropriate cookies, caching headers and the accept header will be set to `application/json`. This fetch does not use service workers.

```js
{
  "@context": "http://schema.org",
  "@type": "CompleteDataFeed",
  "dataFeedElement": [
    { <MEDIA FEED OBJECT 01> },
    { <MEDIA FEED OBJECT 02> },
    { <MEDIA FEED OBJECT 03> },
    ...
  ]
  "provider": {
    "@type": "Organization",
    "member": {
      "@type": "Person",
      "email": "beccahughes@chromium.org",
      "name": "Becca Hughes",
      "image": "https://www.example.org/profile_pic.jpg"
    },
    "name": "Media Site",
    "logo": [
      {
        "@type": "ImageObject",
        "width": 336,
        "height": 188,
        "url": "https://beccahughes.github.io/logo-dark-title-transparent.png",
        "additionalProperty": {
          "@type": "PropertyValue",
          "name": "contentAttributes",
          "value": ["forDarkBackground", "hasTitle", "transparentBackground"]
        }
      },
      {
        "@type": "ImageObject",
        "width": 336,
        "height": 188,
        "url": "https://beccahughes.github.io/logo-light-title-transparent.png",
        "additionalProperty": {
          "@type": "PropertyValue",
          "name": "contentAttributes",
          "value": ["forLightBackground", "hasTitle", "transparentBackground"]
        }
      }
    ]
  },
  "additionalProperty": {
    "@type": "PropertyValue",
    "name": "cookieNameFilter",
    "value": "LOGIN"
  }
}
```

The site should return a schema.org [CompleteDataFeed](https://schema.org/CompleteDataFeed). This has a [provider](https://schema.org/provider) property that uses [Organization](https://schema.org/Organization) to describe the site that publishes the feed. This can be used by the user agent to show the name and logo of the site. The [member](https://schema.org/member) field on Organization can contain a [Person](https://schema.org/Person) that contains the details about the currently logged in user. This should contain the name and optionally the email and/or image of the user. The user agent can show this to the user to identifier which account the recommendations are coming from. This is especially important if the media site supports multiple profiles.

The feed will contain a number of media items that are recommendations relevant to the logged-in user or generic if the user is not logged in. These are stored in the `dataFeedElement` property and should all have a unique `@id` which can be the URL of the content. The media items should be one of the following types:

1.  [Thing > CreativeWork > MediaObject > VideoObject](https://schema.org/VideoObject)
    
2.  [Thing > CreativeWork > Movie](https://schema.org/Movie)
    
3.  [Thing > CreativeWork > CreativeWorkSeries > TVSeries](https://schema.org/TVSeries)

There are some examples in the sections below about how these media items can be used for different use cases.

A site should make use of appropriate [Cache-Control](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control) headers and return a `304 Not Modified` response if the content has not changed. The [Cache-Control](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control) and [Expires](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Expires) headers can be used to suggest to the user agent about when a feed should be re-fetched. The user agent will make a decision about how often to fetch the feed based on a minimum/maximum value and based on user interaction.

### A site recommends a generic video to a user

If a site wishes to recommend a generic video to a user they can use the [VideoObject](https://schema.org/VideoObject) media item type. The URL to watch the video is stored in the [potentialAction](https://schema.org/potentialAction) property.
  
```js
{
  "@context": "http://schema.org/",
  "@type": "VideoObject",
  "@id": "https://example.org/video",
  "author": {
    "@type": "Person",
    "name": "Test User",
    "url": "https://example.org/testuser"
  },
  "datePublished": "2020-01-27",
  "duration": "PT5M49S",
  "genre": "Animated Shorts",
  "image": {
    "@type": "ImageObject",
    "width": 360,
    "height": 480,
    "url": "https://example.org/video_thumbnail.png",
    "additionalProperty": {
      "@type": "PropertyValue",
      "name": "contentAttributes",
      "value": ["iconic", "poster", "hasTitle"]
    }
  },
  "interactionStatistic": {
    "@type": "InteractionCounter",
    "interactionType": "http://schema.org/WatchAction",
    "userInteractionCount": "4356"
  },
  "isFamilyFriendly": "http://schema.org/True",
  "name": "Big Buck Bunny",
  "potentialAction": {
    "@type": "WatchAction",
    "target": "https://example.org/watch/video"
  }
}
```

### A site recommends a media item for a user to continue watching

A user agent might want to display content for the user to continue watching. In this case the media item should contain a [WatchAction](http://schema.org/WatchAction) in the `potentialAction` property. In order to be displayed as continue watching by the user agent the media item must have a duration and the `potentialAction` should contain the following properties:

1.  `actionStatus` set to [ActiveActionStatus](http://schema.org/ActiveActionStatus)
    
2.  `startTime` set to the time offset to start playing in the media item

If the media item url will not resume playback at the correct time, then the `target` of the action can be set to a URL that the user agent will navigate to, to resume playback.

```js
{
  "@context": "http://schema.org/",
  "@type": "VideoObject",
  "@id": "https://example.org/video",
  "author": {
    "@type": "Person",
    "name": "Test User",
    "url": "https://example.org/testuser"
  },
  "datePublished": "2020-01-27",
  "duration": "PT5M49S",
  "genre": "Animated Shorts",
  "image": {
    "@type": "ImageObject",
    "width": 360,
    "height": 480,
    "url": "https://example.org/video_thumbnail.png",
    "additionalProperty": {
      "@type": "PropertyValue",
      "name": "contentAttributes",
      "value": ["sceneStill", "background", "noTitle"]
    }
  },
  "interactionStatistic": {
    "@type": "InteractionCounter",
    "interactionType": "http://schema.org/WatchAction",
    "userInteractionCount": "4356"
  },
  "isFamilyFriendly": "http://schema.org/True",
  "name": "Big Buck Bunny",
  "potentialAction": {
    "@type": "WatchAction",
    "actionStatus": "http://schema.org/ActiveActionStatus",
    "startTime": "00:00:10",
    "target": "https://example.org/video?time=10s"
  }
}
```

### A site recommends a TV episode / season for a user to play next

In a TV Series it is common for a user to be approaching the end of a TV Episode and for the site to recommend for a user to play the next episode. For a site to do this they can use the [TVSeries](https://schema.org/TVSeries) media item. This should embed a TVEpisode that has either an [ActiveActionStatus](https://schema.org/ActiveActionStatus) `WatchAction` that denotes the user is close to the end, or a [CompletedActionStatus](https://schema.org/CompletedActionStatus) `WatchAction` that denotes the user has finished watching this episode.

The user agent will then find the next episode in this series that should have a [PotentialActionStatus](https://schema.org/PotentialActionStatus) `WatchAction` to denote the user has not started watching this episode. The episodes should have an `episodeNumber` set and the next episode should have a number that is the current episode + 1.

```js
{
  "@context": "http://schema.org/",
  "@type": "TVSeries",
  "@id": "https://example.org/example-tv-series",
  "genre": "Documentaries",
  "image": {
    "@type": "ImageObject",
    "width": 360,
    "height": 480,
    "url": "https://example.org/video_thumbnail.png",
    "additionalProperty": {
      "@type": "PropertyValue",
      "name": "contentAttributes",
      "value": ["iconic", "poster", "hasTitle"]
    }
  },
  "isFamilyFriendly": "http://schema.org/True",
  "name": "Cool TV Series",
  "containsSeason": {
    "@type": "TVSeason",
    "numberOfEpisodes": 10,
    "episode": [{
      "@type": "TVEpisode",
      "@id": "https://example.org/example-tv-series?e=1&s=1",
      "duration": "PT5M49S",
      "episodeNumber": 1,
      "potentialAction": {
        "@type": "WatchAction",
        "actionStatus": "http://schema.org/ActiveActionStatus",
        "startTime": "00:00:10",
        "target": "https://example.org/video?time=10s"
      },
      "image": {
        "@type": "ImageObject",
        "width": 360,
        "height": 480,
        "url": "https://example.org/tv_s1_e1_thumbnail.png",
        "additionalProperty": {
          "@type": "PropertyValue",
          "name": "contentAttributes",
          "value": ["sceneStill", "background", "noTitle"]
        }
       },
       "name": "TV Episode 1 Name"
      },
    }, {
      "@type": "TVEpisode",
      "@id": "https://example.org/example-tv-series?e=2&s=1",
      "episodeNumber": 2,
      "potentialAction": {
        "@type": "WatchAction",
        "actionStatus": "http://schema.org/PotentialActionStatus"
      },
      "image": {
        "@type": "ImageObject",
        "width": 360,
        "height": 480,
        "url": "https://example.org/tv_s1_e2_thumbnail.png",
        "additionalProperty": {
          "@type": "PropertyValue",
          "name": "contentAttributes",
          "value": ["sceneStill", "background", "noTitle"]
        }
       },
       "name": "TV Episode 2 Name"
      },
    },
    "seasonNumber": 1
  }] 
}
```

Below is an example where a user has finished watching the first season and the media feed is recommending the user start the next season. In this example the `seasonNumber` must be specified and the next season should have a number that is the current season number +1. The current season should also have a `numberOfEpisodes` that is equal to the current episode number.

```js
{
  "@context": "http://schema.org/",
  "@type": "TVSeries",
  "@id": "https://example.org/example-tv-series",
  "genre": "Documentaries",
  "image": {
    "@type": "ImageObject",
    "width": 360,
    "height": 480,
    "url": "https://example.org/video_thumbnail.png",
    "additionalProperty": {
      "@type": "PropertyValue",
      "name": "contentAttributes",
      "value": ["iconic", "poster", "hasTitle"]
    }
  },
  "isFamilyFriendly": "http://schema.org/True",
  "name": "Cool TV Series",
  "containsSeason": [{
    "@type": "TVSeason",
    "numberOfEpisodes": 10,
    "episode": {
      "@type": "TVEpisode",
      "@id": "https://example.org/example-tv-series?e=10&s=1",
      "episodeNumber": 10,
      "potentialAction": {
        "@type": "WatchAction",
        "actionStatus": "http://schema.org/CompletedActionStatus"
      },
      "image": {
        "@type": "ImageObject",
        "width": 360,
        "height": 480,
        "url": "https://example.org/tv_s1_e10_thumbnail.png",
        "additionalProperty": {
          "@type": "PropertyValue",
          "name": "contentAttributes",
          "value": ["sceneStill", "background", "noTitle"]
        }
       },
       "name": "TV Episode 10 Name"
      },
    },
    "seasonNumber": 1
  }, {
    "@type": "TVSeason",
    "numberOfEpisodes": 10,
    "episode": {
      "@type": "TVEpisode",
      "@id": "https://example.org/example-tv-series?e=1&s=2",
      "episodeNumber": 1,
      "potentialAction": {
        "@type": "WatchAction",
        "actionStatus": "http://schema.org/PotentialActionStatus"
      },
      "image": {
        "@type": "ImageObject",
        "width": 360,
        "height": 480,
        "url": "https://example.org/tv_s2_e1_thumbnail.png",
        "additionalProperty": {
          "@type": "PropertyValue",
          "name": "contentAttributes",
          "value": ["sceneStill", "background", "noTitle"]
        }
       },
       "name": "TV Episode 10 Name"
      },
    },
    "seasonNumber": 2
  }] 
}
```

### A site recommends a TV series to a user

If a site wishes to recommend a TV series to a user they can use the [TVSeries](https://schema.org/TVSeries) media item type. It is important to use the TVSeries type instead of the VideoObject type because a user agent might want to filter and group content into Movies, TV shows, etc.

```js
{
  "@context": "http://schema.org/",
  "@type": "TVSeries",
  "@id": "https://example.org/tv-series",
  "genre": "Documentaries",
  "image": {
    "@type": "ImageObject",
    "width": 360,
    "height": 480,
    "url": "https://example.org/video_thumbnail.png",
    "additionalProperty": {
      "@type": "PropertyValue",
      "name": "contentAttributes",
      "value": ["iconic", "poster", "hasTitle"]
    }
  },
  "isFamilyFriendly": "http://schema.org/True",
  "name": "Cool TV Series",
  "potentialAction": {
    "@type": "WatchAction",
    "target": "https://example.org/watch/tv-series"
  }
}
```

### A site recommends some live video content to a user

If a site wishes to recommend some media that is being broadcast or streamed they can add a [BroadcastEvent](https://schema.org/BroadcastEvent) to the feed. This is a special type that describes a broadcast schedule or a live stream and embeds the media item being broadcast in the `workPerformed` property.

```js
{
  "@context": "https://schema.org/",
  "@type": "BroadcastEvent",
  "@id": "https://example.org/event",
  "isLiveBroadcast": "http://schema.org/True",
  "startDate": "2020-01-28T06:00:00+0000",
  "endDate": "2020-01-28T07:00:00+0000",
  "potentialAction": {
    "@type": "WatchAction",
    "target": "https://example.org/watch/video"
  },
  "workPerformed": {
    "@context": "https://schema.org/",
    "@type": "VideoObject",
    "@id": "https://example.org/video",
    "author": {
      "@type": "Person",
      "name": "Test User",
      "url": "https://example.org/testuser"
    },
    "datePublished": "2020-01-27",
    "duration": "PT5M49S",
    "genre": "Animated Shorts",
    "image": {
      "@type": "ImageObject",
      "width": 360,
      "height": 480,
      "url": "https://example.org/video_thumbnail.png",
      "additionalProperty": {
        "@type": "PropertyValue",
        "name": "contentAttributes",
        "value": ["iconic", "poster", "hasTitle"]
      }
    },
    "interactionStatistic": {
      "@type": "InteractionCounter",
      "interactionType": "http://schema.org/WatchAction",
      "userInteractionCount": "4356"
    },
    "isFamilyFriendly": "http://schema.org/True",
    "name": "Big Buck Bunny"
  }
}
```

### A site recommends a movie to a user

If a site wishes to recommend a movie to a user they can use the [Movie](https://schema.org/Movie) media item type. It is important to use the Movie type instead of the VideoObject type because a user agent might want to filter and group content into Movies, TV shows, etc.

```js
{
  "@context": "http://schema.org/",
  "@type": "Movie",
  "@id": "https://example.org/dream",
  "contentRating": {
    "@type": "Rating",
    "author": "MPAA",
    "ratingValue": "PG-13"
  }
  "datePublished": "2020-01-27",
  "duration": "PT5M49S",
  "genre": "Animated",
  "image": {
    "@type": "ImageObject",
    "width": 360,
    "height": 480,
    "url": "https://example.org/movie_thumbnail.png",
    "additionalProperty": {
      "@type": "PropertyValue",
      "name": "contentAttributes",
      "value": ["iconic", "poster", "hasTitle"]
    }
  },
  "isFamilyFriendly": "http://schema.org/True",
  "name": "Dream",
  "potentialAction": {
    "@type": "WatchAction",
    "target": "https://example.org/watch/movie"
  }
}
```
 
### Recommendations around properties

It is **required** that all media items should have the following properties:

1. `@id` - this should be unique across all ids in the feed
    
2. `name` - the name of the content
    
3. `author` - the author / creator of the content (only for VideoObject)
    
4. `datePublished` - the timestamp the content was published
    
5. `duration` - the duration of the content (only required for VideoObject and Movie, also optional for videos that are live)
    
6. `image` - artwork to be displayed by the user agent. These should be [ImageObject](https://schema.org/ImageObject). This allows the user agent to pick the right artwork based on the size

7. `potentialAction` - the action to watch the media. If the object is a [TVSeries](https://schema.org/TVSeries) and has an embedded [TVEpisode](https://schema.org/TVEpisode) then the `potentialAction` is only required on the [TVEpisode](https://schema.org/TVEpisode). The `potentialAction` is also not required on items that are embedded in a broadcast event.
    
It is *suggested* that the media items have the following properties:

1. `interactionStatistic`- this count of any interactions on the content. The recommended interactions to support are [WatchAction](https://schema.org/WatchAction), [LikeAction](https://schema.org/LikeAction) and [DislikeAction](https://schema.org/DislikeAction)
    
2. `contentRating` - the content rating of the content (e.g. PG-18)
    
3. `genre` - the genre of the content

4. `identifier` - the identifiers of the content in third party systems

5. `isFamilyFriendly` - if the content is family friendly

WatchActions should be present in the `potentialAction` property and have the following requirements:

1. The `actionStatus` field **is recommended**. If present, it must be one of `ActiveActionStatus`, `PotentialActionStatus` or `CompletedActionStatus`
    
2. If the `actionStatus` is `ActiveActionStatus` the `startTime` **must be** present
    
3. A `target` URL **must be** present

If an object is a TV season the following properties **are required**:

1. `seasonNumber` - the number of the season in the TV series, should be sequential
    
2. `numberOfEpisodes` - the total number of episodes a series has

If an object is a TV episode the following properties **are required**:

1. `episodeNumber` - the number of the episode in the TV series or season, should be sequential

2. `duration` - the duration of the content

3. `name` - the name of the episode

4. `image` - artwork to be displayed by the user agent. These should be [ImageObject](https://schema.org/ImageObject). This allows the user agent to pick the right artwork based on the size

5. `potentialAction` - the action to watch the media.

It is *suggested* that the TV episode items have the following properties:

1. `identifier` - the identifiers of the content in third party systems

If the `image` is an ImageObject the following properties **are required**:

1. `width` - the width of the image in px
    
2. `height` - the height of the image in px
    
3. `embedUrl` or `url` - the URL to embed the content

4. `additionalProperty` - this can contain a `contentAttributes` property which contains an array of different attributes that tell the user agent about the contents of the image
    
It is **required** that all media feed broadcast events have the following properties:

1. `@id` - this should be unique across all ids in the feed

2. `isLiveBroadcast` **must be** set to true or false
    
3. `startDate` **must be** present and set to the datetime the broadcast started
    
4. `endDate` is optional *if known*

5. `potentialAction` - the action to watch the media

6. `workPerformed` - contains an embedded media feed item that is the media being broadcast
    
**Any media items that do not have the required properties, or properties that are set to non-standard values will be ignored.**

In schema.org any object can have a number of [identifier](https://schema.org/identifier) properties. These are [PropertyValue](https://schema.org/PropertyValue) objects that have a `propertyID` and a `value` property. There is no defined list of property identifiers and they are left up to the implementation. The media items can have a number of identifiers set by the site and an example is below:

```js
{
  "@context": "http://schema.org/",
  "@type": "Movie",
  "@id": "https://example.org/dream",
  ...

  "identifier": {
    "@type": "PropertyValue",
    "propertyID": "OTHER_ID_TYPE_A",
    "value": "456789876"
  },
  "identifier": {
    "@type": "PropertyValue",
    "propertyID": "OTHER_ID_TYPE_B",
    "value": "ZZZZZZ456789876"
  }

  ...
}
```
  
A list of known identifiers is [available here](third-party-identifiers.md).

### Alternatives Considered

## JSON Feeds

Pros:

  - JSON based (easier to use and browsers have native support)
  
  - Extendible
 
Cons: 

  - Media providers are not familiar with JSON feeds (they are with schema.org)
  
  - The image URL is fixed and not extensible and we need to support additional data on an image such as width, height and content attributes

## MediaRSS

Pros:

  - XML is supported by browsers
 
Cons: 

  - Media providers are not familiar with JSON feeds (they are with schema.org)
  
  - Only supports generic media types with limited metadata and does not provide us with the level of detail required, such as continue watching information or detailed images
  
  - Browsers do not have native support for MediaRSS

## Atom

Pros:

  - Relatively common
 
Cons: 

  - Nobody uses RDFa in Atom which is going to make it harder for developers
  
  - Browsers do not have native support for Atom
