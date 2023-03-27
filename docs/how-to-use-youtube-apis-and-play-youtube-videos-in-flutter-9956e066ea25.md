# 如何使用 YouTube APIs 并在 Flutter 中播放 YouTube 视频

> 原文：<https://itnext.io/how-to-use-youtube-apis-and-play-youtube-videos-in-flutter-9956e066ea25?source=collection_archive---------3----------------------->

在这篇文章中，我们将讨论如何使用 YouTube 的 API 和在 Flutter 中播放 YouTube 视频。

![](img/dead09c4a05ab6f061cab8b0e6823eac.png)

在 Flutter 中播放 Youtube 视频

# 观看视频教程

在 Flutter 中播放 Youtube 视频

# **添加依赖关系**

打开你的 Flutter 项目中的 ***pubspec.yaml*** 文件，添加下面的依赖项

```
http: ^0.12.0+2
youtube_player_flutter: ^7.0.0+7
cached_network_image: ^2.2.0+1
```

现在，让我们创建两个屏幕

1.  主屏幕
2.  视频播放器屏幕

在主屏幕上，我们将显示视频列表。

# API 密钥

我们需要 API 密钥来访问 YouTube APIs。

为此，我们需要去 https://console.developers.google.com/

*创建一个新项目，*

1.  *点击创建项目*
2.  *完成后，点击仪表板、启用的 API，找到 YouTube API V3*
3.  *点击启用*
4.  *创建凭证，选择 Youtube API V3*
5.  *选择平台*
6.  *选择‘公共数据’*
7.  *点击“我需要什么凭证？”*
8.  *点击完成*
9.  *复制 API 密钥。*

让我们将这个 api 密钥复制到我们的 Flutter 项目中。

```
String API_KEY = 'AIzaSyBGj_Duj__ivCxJ2ya3ilkVfEzX1ZSRlpE';
```

**获取频道信息**

要获取频道信息，我们可以使用下面的 youtube API

[](https://developers.google.com/youtube/v3/docs/channels/list) [## 频道:列表| YouTube 数据 API |谷歌开发者

### 返回与请求条件匹配的零个或多个通道资源的集合。配额影响:对此的呼吁…

developers.google.com](https://developers.google.com/youtube/v3/docs/channels/list) 

让我们创建一个服务类来调用这些 API。

创建一个名为“ *Services.dart* 的类，调用上面的 API。

获取某个 YouTube 频道的频道 ID。

下面是我的频道 ID。

[https://www.youtube.com/channel/**UC 5 lbdurzjb 0 IRR-FTbjWN1A**](https://www.youtube.com/channel/UC5lbdURzjB0irr-FTbjWN1A)

```
static const CHANNEL_ID = 'UC5lbdURzjB0irr-FTbjWN1A';
  static const _baseUrl = '[www.googleapis.com'](http://www.googleapis.com%27/);static Future<ChannelInfo> getChannelInfo() async {
    Map<String, String> parameters = {
      'part': 'snippet,contentDetails,statistics',
      'id': CHANNEL_ID,
      'key': Constants.API_KEY,
    };
    Map<String, String> headers = {
      HttpHeaders.contentTypeHeader: 'application/json',
    };
    Uri uri = Uri.https(
      _baseUrl,
      '/youtube/v3/channels',
      parameters,
    );
    Response response = await http.get(uri, headers: headers);
    // print(response.body);
    ChannelInfo channelInfo = channelInfoFromJson(response.body);
    return channelInfo;
}
```

**ChannelInfo** 上述 API 的模型类。

```
import 'dart:convert';ChannelInfo channelInfoFromJson(String str) =>
    ChannelInfo.fromJson(json.decode(str));String channelInfoToJson(ChannelInfo data) => json.encode(data.toJson());class ChannelInfo {
  ChannelInfo({
    this.kind,
    this.etag,
    this.pageInfo,
    this.items,
  });String kind;
  String etag;
  PageInfo pageInfo;
  List<Item> items;factory ChannelInfo.fromJson(Map<String, dynamic> json) => ChannelInfo(
        kind: json["kind"],
        etag: json["etag"],
        pageInfo: PageInfo.fromJson(json["pageInfo"]),
        items: List<Item>.from(json["items"].map((x) => Item.fromJson(x))),
      );Map<String, dynamic> toJson() => {
        "kind": kind,
        "etag": etag,
        "pageInfo": pageInfo.toJson(),
        "items": List<dynamic>.from(items.map((x) => x.toJson())),
      };
}class Item {
  Item({
    this.kind,
    this.etag,
    this.id,
    this.snippet,
    this.contentDetails,
    this.statistics,
  });String kind;
  String etag;
  String id;
  Snippet snippet;
  ContentDetails contentDetails;
  Statistics statistics;factory Item.fromJson(Map<String, dynamic> json) => Item(
        kind: json["kind"],
        etag: json["etag"],
        id: json["id"],
        snippet: Snippet.fromJson(json["snippet"]),
        contentDetails: ContentDetails.fromJson(json["contentDetails"]),
        statistics: Statistics.fromJson(json["statistics"]),
      );Map<String, dynamic> toJson() => {
        "kind": kind,
        "etag": etag,
        "id": id,
        "snippet": snippet.toJson(),
        "contentDetails": contentDetails.toJson(),
        "statistics": statistics.toJson(),
      };
}class ContentDetails {
  ContentDetails({
    this.relatedPlaylists,
  });RelatedPlaylists relatedPlaylists;factory ContentDetails.fromJson(Map<String, dynamic> json) => ContentDetails(
        relatedPlaylists: RelatedPlaylists.fromJson(json["relatedPlaylists"]),
      );Map<String, dynamic> toJson() => {
        "relatedPlaylists": relatedPlaylists.toJson(),
      };
}class RelatedPlaylists {
  RelatedPlaylists({
    this.likes,
    this.favorites,
    this.uploads,
    this.watchHistory,
    this.watchLater,
  });String likes;
  String favorites;
  String uploads;
  String watchHistory;
  String watchLater;factory RelatedPlaylists.fromJson(Map<String, dynamic> json) =>
      RelatedPlaylists(
        likes: json["likes"],
        favorites: json["favorites"],
        uploads: json["uploads"],
        watchHistory: json["watchHistory"],
        watchLater: json["watchLater"],
      );Map<String, dynamic> toJson() => {
        "likes": likes,
        "favorites": favorites,
        "uploads": uploads,
        "watchHistory": watchHistory,
        "watchLater": watchLater,
      };
}class Snippet {
  Snippet({
    this.title,
    this.description,
    this.publishedAt,
    this.thumbnails,
    this.localized,
    this.country,
  });String title;
  String description;
  DateTime publishedAt;
  Thumbnails thumbnails;
  Localized localized;
  String country;factory Snippet.fromJson(Map<String, dynamic> json) => Snippet(
        title: json["title"],
        description: json["description"],
        publishedAt: DateTime.parse(json["publishedAt"]),
        thumbnails: Thumbnails.fromJson(json["thumbnails"]),
        localized: Localized.fromJson(json["localized"]),
        country: json["country"],
      );Map<String, dynamic> toJson() => {
        "title": title,
        "description": description,
        "publishedAt": publishedAt.toIso8601String(),
        "thumbnails": thumbnails.toJson(),
        "localized": localized.toJson(),
        "country": country,
      };
}class Localized {
  Localized({
    this.title,
    this.description,
  });String title;
  String description;factory Localized.fromJson(Map<String, dynamic> json) => Localized(
        title: json["title"],
        description: json["description"],
      );Map<String, dynamic> toJson() => {
        "title": title,
        "description": description,
      };
}class Thumbnails {
  Thumbnails({
    this.thumbnailsDefault,
    this.medium,
    this.high,
  });Default thumbnailsDefault;
  Default medium;
  Default high;factory Thumbnails.fromJson(Map<String, dynamic> json) => Thumbnails(
        thumbnailsDefault: Default.fromJson(json["default"]),
        medium: Default.fromJson(json["medium"]),
        high: Default.fromJson(json["high"]),
      );Map<String, dynamic> toJson() => {
        "default": thumbnailsDefault.toJson(),
        "medium": medium.toJson(),
        "high": high.toJson(),
      };
}class Default {
  Default({
    this.url,
    this.width,
    this.height,
  });String url;
  int width;
  int height;factory Default.fromJson(Map<String, dynamic> json) => Default(
        url: json["url"],
        width: json["width"],
        height: json["height"],
      );Map<String, dynamic> toJson() => {
        "url": url,
        "width": width,
        "height": height,
      };
}class Statistics {
  Statistics({
    this.viewCount,
    this.commentCount,
    this.subscriberCount,
    this.hiddenSubscriberCount,
    this.videoCount,
  });String viewCount;
  String commentCount;
  String subscriberCount;
  bool hiddenSubscriberCount;
  String videoCount;factory Statistics.fromJson(Map<String, dynamic> json) => Statistics(
        viewCount: json["viewCount"],
        commentCount: json["commentCount"],
        subscriberCount: json["subscriberCount"],
        hiddenSubscriberCount: json["hiddenSubscriberCount"],
        videoCount: json["videoCount"],
      );Map<String, dynamic> toJson() => {
        "viewCount": viewCount,
        "commentCount": commentCount,
        "subscriberCount": subscriberCount,
        "hiddenSubscriberCount": hiddenSubscriberCount,
        "videoCount": videoCount,
      };
}class PageInfo {
  PageInfo({
    this.resultsPerPage,
  });int resultsPerPage;factory PageInfo.fromJson(Map<String, dynamic> json) => PageInfo(
        resultsPerPage: json["resultsPerPage"],
      );Map<String, dynamic> toJson() => {
        "resultsPerPage": resultsPerPage,
      };
}
```

现在让我们在主屏幕中使用这个 API。

```
ChannelInfo _channelInfo;_getChannelInfo() async {
    _channelInfo = await Services.getChannelInfo();
    _item = _channelInfo.items[0];
    _playListId = _item.contentDetails.relatedPlaylists.uploads;
    print('_playListId $_playListId');
    await _loadVideos();
    setState(() {
      _loading = false;
    });
 }
```

**让我们构建渠道信息视图。**

_item.snippet.title. —获取频道标题

_ item . snippet . thumbnails . medium . URL-获取频道缩略图。中等分辨率。

```
_buildInfoView() {
    return _loading
        ? CircularProgressIndicator()
        : Container(
            padding: EdgeInsets.all(20.0),
            child: Card(
              child: Padding(
                padding: const EdgeInsets.all(10.0),
                child: Row(
                  children: [
                    CircleAvatar(
                      backgroundImage: CachedNetworkImageProvider(
                        _item.snippet.thumbnails.medium.url,
                      ),
                    ),
                    SizedBox(width: 20),
                    Expanded(
                      child: Text(
                        _item.snippet.title,
                        style: TextStyle(
                          fontSize: 20,
                          fontWeight: FontWeight.w400,
                        ),
                      ),
                    ),
                    Text(_item.statistics.videoCount),
                    SizedBox(width: 20),
                  ],
                ),
              ),
            ),
          );
  }
```

![](img/cf935522ed49c8b6ec1398b25c300a1c.png)

频道信息

# 获取视频列表

要获得视频列表，我们需要使用下面的 YouTube API

[](https://developers.google.com/youtube/v3/docs/playlistItems/list) [## PlaylistItems: list | YouTube 数据 API | Google 开发者

### string 此参数只能在正确授权的请求中使用。注意:此参数专用于…

developers.google.com](https://developers.google.com/youtube/v3/docs/playlistItems/list) 

让我们编写获取视频列表的服务。

```
static Future<VideosList> getVideosList(
      {String playListId, String pageToken}) async {
    Map<String, String> parameters = {
      'part': 'snippet',
      'playlistId': playListId,
      'maxResults': '8',
      'pageToken': pageToken,
      'key': Constants.API_KEY,
    };
    Map<String, String> headers = {
      HttpHeaders.contentTypeHeader: 'application/json',
    };
    Uri uri = Uri.https(
      _baseUrl,
      '/youtube/v3/playlistItems',
      parameters,
    );
    Response response = await http.get(uri, headers: headers);
    // print(response.body);
    VideosList videosList = videosListFromJson(response.body);
    return videosList;
}
```

上述 API 的**视频列表**模型

```
import 'dart:convert';VideosList videosListFromJson(String str) =>
    VideosList.fromJson(json.decode(str));String videosListToJson(VideosList data) => json.encode(data.toJson());class VideosList {
  VideosList({
    this.kind,
    this.etag,
    this.nextPageToken,
    this.videos,
    this.pageInfo,
  });String kind;
  String etag;
  String nextPageToken;
  List<VideoItem> videos;
  PageInfo pageInfo;factory VideosList.fromJson(Map<String, dynamic> json) => VideosList(
        kind: json["kind"],
        etag: json["etag"],
        nextPageToken: json["nextPageToken"],
        videos: List<VideoItem>.from(
            json["items"].map((x) => VideoItem.fromJson(x))),
        pageInfo: PageInfo.fromJson(json["pageInfo"]),
      );Map<String, dynamic> toJson() => {
        "kind": kind,
        "etag": etag,
        "nextPageToken": nextPageToken,
        "items": List<dynamic>.from(videos.map((x) => x.toJson())),
        "pageInfo": pageInfo.toJson(),
      };
}class VideoItem {
  VideoItem({
    this.kind,
    this.etag,
    this.id,
    this.video,
  });String kind;
  String etag;
  String id;
  Video video;factory VideoItem.fromJson(Map<String, dynamic> json) => VideoItem(
        kind: json["kind"],
        etag: json["etag"],
        id: json["id"],
        video: Video.fromJson(json["snippet"]),
      );Map<String, dynamic> toJson() => {
        "kind": kind,
        "etag": etag,
        "id": id,
        "snippet": video.toJson(),
      };
}class Video {
  Video({
    this.publishedAt,
    this.channelId,
    this.title,
    this.description,
    this.thumbnails,
    this.channelTitle,
    this.playlistId,
    this.position,
    this.resourceId,
  });DateTime publishedAt;
  String channelId;
  String title;
  String description;
  Thumbnails thumbnails;
  String channelTitle;
  String playlistId;
  int position;
  ResourceId resourceId;factory Video.fromJson(Map<String, dynamic> json) => Video(
        publishedAt: DateTime.parse(json["publishedAt"]),
        channelId: json["channelId"],
        title: json["title"],
        description: json["description"],
        thumbnails: Thumbnails.fromJson(json["thumbnails"]),
        channelTitle: json["channelTitle"],
        playlistId: json["playlistId"],
        position: json["position"],
        resourceId: ResourceId.fromJson(json["resourceId"]),
      );Map<String, dynamic> toJson() => {
        "publishedAt": publishedAt.toIso8601String(),
        "channelId": channelId,
        "title": title,
        "description": description,
        "thumbnails": thumbnails.toJson(),
        "channelTitle": channelTitle,
        "playlistId": playlistId,
        "position": position,
        "resourceId": resourceId.toJson(),
      };
}class ResourceId {
  ResourceId({
    this.kind,
    this.videoId,
  });String kind;
  String videoId;factory ResourceId.fromJson(Map<String, dynamic> json) => ResourceId(
        kind: json["kind"],
        videoId: json["videoId"],
      );Map<String, dynamic> toJson() => {
        "kind": kind,
        "videoId": videoId,
      };
}class Thumbnails {
  Thumbnails({
    this.thumbnailsDefault,
    this.medium,
    this.high,
    this.standard,
    this.maxres,
  });Default thumbnailsDefault;
  Default medium;
  Default high;
  Default standard;
  Default maxres;factory Thumbnails.fromJson(Map<String, dynamic> json) => Thumbnails(
        thumbnailsDefault: Default.fromJson(json["default"]),
        medium: Default.fromJson(json["medium"]),
        high: Default.fromJson(json["high"]),
        standard: null == json["standard"]
            ? null
            : Default.fromJson(json["standard"]),
        maxres:
            null == json["maxres"] ? null : Default.fromJson(json["maxres"]),
      );Map<String, dynamic> toJson() => {
        "default": thumbnailsDefault.toJson(),
        "medium": medium.toJson(),
        "high": high.toJson(),
        "standard": standard.toJson(),
        "maxres": maxres.toJson(),
      };
}class Default {
  Default({
    this.url,
    this.width,
    this.height,
  });String url;
  int width;
  int height;factory Default.fromJson(Map<String, dynamic> json) => Default(
        url: json["url"],
        width: json["width"],
        height: json["height"],
      );Map<String, dynamic> toJson() => {
        "url": url,
        "width": width,
        "height": height,
      };
}class PageInfo {
  PageInfo({
    this.totalResults,
    this.resultsPerPage,
  });int totalResults;
  int resultsPerPage;factory PageInfo.fromJson(Map<String, dynamic> json) => PageInfo(
        totalResults: json["totalResults"],
        resultsPerPage: json["resultsPerPage"],
      );Map<String, dynamic> toJson() => {
        "totalResults": totalResults,
        "resultsPerPage": resultsPerPage,
      };
}
```

所以让我们称之为 API

```
_loadVideos() async {
    VideosList tempVideosList = await Services.getVideosList(
      playListId: _playListId,
      pageToken: _nextPageToken,
    );
    _nextPageToken = tempVideosList.nextPageToken;
    _videosList.videos.addAll(tempVideosList.videos);
    print('videos: ${_videosList.videos.length}');
    print('_nextPageToken $_nextPageToken');
    setState(() {});
}
```

在 build 方法中，我们将在 ListView 中显示它

```
ListView.builder(
  controller: _scrollController,
  itemCount: _videosList.videos.length,
  itemBuilder: (context, index) {
    VideoItem videoItem = _videosList.videos[index];
    return InkWell(
      onTap: () async {
        Navigator.push(context,
            MaterialPageRoute(builder: (context) {
          return VideoPlayerScreen(
            videoItem: videoItem,
          );
        }));
      },
      child: Container(
        padding: EdgeInsets.all(20.0),
        child: Row(
          children: [
            CachedNetworkImage(
              imageUrl: videoItem
                  .video.thumbnails.thumbnailsDefault.url,
            ),
            SizedBox(width: 20),
            Flexible(child: Text(videoItem.video.title)),
          ],
        ),
      ),
    );
  },
),
```

![](img/6697611f7980079b00153e94930e0da2.png)

Youtube API 中的视频列表

# **视频播放器屏幕**

现在我们有了每个视频项，每个视频项都有一个 videoId，可以用来加载视频。

视频播放器屏幕将从上面接受 VideoItem 模型并播放视频。

```
import 'package:flutter/material.dart';
import 'package:flutter_demos/models/viideos_list.dart';
import 'package:youtube_player_flutter/youtube_player_flutter.dart';class VideoPlayerScreen extends StatefulWidget {
  //
  VideoPlayerScreen({this.videoItem});
  final VideoItem videoItem;[@override](http://twitter.com/override)
  _VideoPlayerScreenState createState() => _VideoPlayerScreenState();
}class _VideoPlayerScreenState extends State<VideoPlayerScreen> {
  //
  YoutubePlayerController _controller;
  bool _isPlayerReady;[@override](http://twitter.com/override)
  void initState() {
    super.initState();
    _isPlayerReady = false;
    _controller = YoutubePlayerController(
      initialVideoId: widget.videoItem.video.resourceId.videoId,
      flags: YoutubePlayerFlags(
        mute: false,
        autoPlay: true,
      ),
    )..addListener(_listener);
  }void _listener() {
    if (_isPlayerReady && mounted && !_controller.value.isFullScreen) {
      //
    }
  }[@override](http://twitter.com/override)
  void deactivate() {
    _controller.pause();
    super.deactivate();
  }[@override](http://twitter.com/override)
  void dispose() {
    _controller.dispose();
    super.dispose();
  }[@override](http://twitter.com/override)
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.videoItem.video.title),
      ),
      body: Container(
        child: YoutubePlayer(
          controller: _controller,
          showVideoProgressIndicator: true,
          onReady: () {
            print('Player is ready.');
            _isPlayerReady = true;
          },
        ),
      ),
    );
  }
}
```

![](img/37d0a196c8528605b80241c340d0b19d.png)

在 Flutter 中播放 YouTube 视频

# 源代码

 [## 比特桶

### 编辑描述

bitbucket.org](https://bitbucket.org/vipinvijayan1987/tutorialprojects/src/YoutubePlayer/FlutterTutorialProjects/flutter_demos/) 

就是这样。

观看我的 YouTube 视频教程，了解更详细的解释。

感谢阅读。

点击查看我的博客[。](http://www.coderzheaven.com/)

如果你觉得这个教程有用，请鼓掌。