---
title: OpenStack 系列（网络）简单概念
tags:
  - openstack
  - network
  - neutron
date: 2022-05-16 19:58:59
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

> 本篇博文翻译自 OpenStack 官网 [^1]

讲解 OpenStack 网络基本概念

<!--more-->

OpenStack 网络（neutron）管理着 OpenStack 环境的虚拟网络基础设施（VNI：Virtual Networking Infrastruce）所有网络方面，以及物理网络基础设施（PNI：Physical Networking Infrastructure）访问层面。OpenStack 网络支持租户创建高级的虚拟网络特性，诸如防火墙、虚拟个人网络（VPN）。

网络编排支持网络、子网和路由器等对象的抽象。每种抽象都能在功能性上都能类比于物理部分，网络包含子网，路由器能在不同子网和网络间引导流量。

任何已知网络都会安装至少一个外部网络。不像其它网络，这外部网络不仅仅是一个虚拟化定义的网络。反而，它表示一个进入一系列物理网络，访问 OpenStack 外部网络的视角。

相对于外部网络，任何联网都是有不止一个内部网络。这些软件定义的网络让虚机直接连接。当然，只有当虚机在任何内部网络或子网通过网络接口类似路由器连接，才能真正通过网络连接访问虚机。

要想在网络外部去访问虚机，各网络间的路由器是必需的。每个路由器都会有一个网关能连接外部网络及一个甚至更多网络接口连接内部网络。就像物理路由器，同一路由器下不同子网的机器能互访，机器也能通过路由器的网关访问外部网络。

另外，你可以分配 IP 地址给内部、外部网络的指定端口上。实现子网互联的连接点称为端口。你可以关联外部网络 IP 地址到连接虚机的端口上。通过这种方式，外部网络即可访问虚机。

网络同样支持安全组。安全组支持管理员以分组方式定义防火墙规则。一台虚机可以属于一个甚至多个安全组，在安全组里应用规则能阻止或放开虚机的指定端口、端口列表或流量类型。

网络中每个插件都有相应的概念。相比运营 VNI 和 OpenStack 环境，理解这些概念帮助组装网络反而更显重要。所有的网络安装步骤都是采用核心插件和安全组插件（或无需操作）。

[^1]:https://docs.openstack.org/neutron/latest/install/concepts.html
