---

title: pick-up-blog
date: 2022-04-10 17:27:49
tags:
    - journal
---

<!--
  ~ Copyright 2022 kwanhur
  ~
  ~ Licensed under the Apache License, Version 2.0 (the "License");
  ~ you may not use this file except in compliance with the License.
  ~ You may obtain a copy of the License at
  ~
  ~ http://www.apache.org/licenses/LICENSE-2.0
  ~
  ~ Unless required by applicable law or agreed to in writing, software
  ~ distributed under the License is distributed on an "AS IS" BASIS,
  ~ WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  ~ See the License for the specific language governing permissions and
  ~ limitations under the License.
  ~
-->

该博客被遗弃很久了，近期看到 `NexT` 主题比较合心意，决定折腾下，更换了主题，看到的效果还不错。

另一原因是，之前写的分享内容全部在公司内部`wiki`系统，离开后也就留下了；所以决定后续关于技术研究、想法点子等采用`Github Pages`形式分享。

<!--more-->

折腾过程还是遇到点小麻烦：本地预览没问题，发布推送上`Github`后，样式错乱，即使禁用浏览器缓存也不起作用。

经一番翻查，对比样式文件`css/main.css`本地与`Github`请求响应体大小不一致。

手动触发请求`https://$repo.github.io/css/main.css`随即解决。

原因猜测：旧主题所用的样式文件一直被服务器端缓存，经`hexo deploy`推送至`Github`，但`Github Pages`服务并不会更新静态文件缓存。
