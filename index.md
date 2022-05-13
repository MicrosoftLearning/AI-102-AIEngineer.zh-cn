---
title: 联机托管说明
permalink: index.html
layout: home
ms.openlocfilehash: ba3d2a5154e5e9b08e750f7d9ced8b62048dd66d
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/16/2021
ms.locfileid: "137819332"
---
# <a name="content-directory"></a>内容目录

此存储库包含针对 Microsoft 课程 [AI-102 设计和实现 Microsoft Azure AI 解决方案](https://docs.microsoft.com/learn/certifications/courses/ai-102t00)的动手实验室练习以及等效的 [Microsoft Learn 上的自定进度模块](https://aka.ms/AzureLearn_AIEngineer)。 这些练习配合教材提供，你可以使用其描述的方法进行练习。

若要完成这些练习，需要一个 Microsoft Azure 订阅。 如果讲师未提供，可以在 [https://azure.microsoft.com](https://azure.microsoft.com) 注册免费试用版。

## <a name="labs"></a>实验室

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions'" %}
| 模块 | 实验室 |
| --- | --- | 
{% 表示实验室 % 中的活动}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

