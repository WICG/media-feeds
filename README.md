
# Media Feeds

This is the repository for media-feeds. You're welcome to contribute! Let's make the Web rock our socks
off!

[Read The Spec](https://wicg.github.io/media-feeds/)

## Browser Support - Chrome

Chrome is currently experimenting with Media Feeds. For the best experience we recommend that all feeds
have the following logos:

1. A logo that is landscape, suitable for dark backgrounds, has the title and logo for the website and
has a transparent background. It should have the `forDarkBackground`, `hasTitle` and `transparentBackground`
[content attributes](https://wicg.github.io/media-feeds/#dfn-media-logo).

2. A logo that is landscape, suitable for light backgrounds, has the title and logo for the website and
has a transparent background. It should have the `forLightBackground`, `hasTitle` and `transparentBackground`
[content attributes](https://wicg.github.io/media-feeds/#dfn-media-logo).

3. A logo that is landscape, suitable for dark backgrounds, has the title and logo centered, and has
an opaque background. It should have the `forDarkBackground`, `hasTitle` and `centered`
[content attributes](https://wicg.github.io/media-feeds/#dfn-media-logo).

We recommend that the media items have the following images:

1. An image that is landscape poster or banner of the media item. It should have the `iconic` and
`poster` [content attributes](https://wicg.github.io/media-feeds/#dfn-media-content-image). This is
usually the primary image used to display the media item.

2. An image that is landscape frame grab / scene still. It should have the `sceneStill`
[content attribute](https://wicg.github.io/media-feeds/#dfn-media-content-image) and optionally the
`background` attribute if it is certain the image does not contain text. This is recommended for
continue watching scenarios.

3. An image that is landscape, an artistic render of the media item and a background. It should have the
`iconic` and `background` [content attributes](https://wicg.github.io/media-feeds/#dfn-media-content-image).
This is not used at the moment but is recommended for future use.

This might not always be possible for user generated content since there are no guarantees that images
will meet these requirements. In these cases the content attributes are optional and we recommend a
primary image used to display the media item which might not have any attributes and possibly a
`sceneStill` image if available.
