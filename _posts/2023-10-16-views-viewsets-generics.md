---
layout: post
title: "[DRF] DRF의 View Classes - GenericAPIView, GenericViewSet, ViewSet"
subtitle:
categories: Django DRF
tags: []
---
  
django class base로 api를 구현하려다보니, drf에서 제공하는 다양한 모듈이 보였다. View 클래스를 상속받아 클래스를 구현하는 것이 보통이었는데, 각각이 제공하는 기본 기능들을 살펴보고 적당한 것을 도입하려고 살펴봤다. 

![drf-views.png](https://github.com/aohus/aohus.github.io/blob/main/assets/images/posts/drf-views.png)

그림은 아래 문서를 참고했다. 상위 class로 갈수록 기본적으로 제공하는 기능은 적고 자유도가 높다. 

APIView에서 갈라지는 두 클래스를 먼저 살펴보자. 

### GenericAPIView
`GenericAPIView`에 추가된 메서드들은 아래와 같다. 
```python
get_queryset
get_object
get_serializer
get_serializer_context
filter_queryset
paginator
pagenate_queryset
get_paginated_response
```


### ViewSet
ViewSet 자체는 `ViewSetMixin`과 `APIView` 를 상속받고 스스로 추가한 기능은 없는 클래스이다. 
```python
class ViewSet(ViewSetMixin, views.APIView):
    """
    The base ViewSet class does not provide any actions by default.
    """
    pass
```

그럼 `APIView` 에 추가된 기능은 `ViewSetMixin` 만 살펴보면 된다. 
![](https://github.com/aohus/aohus.github.io/blob/main/assets/images/posts/drf-view-mixin.png)
ViewSetMixin은 마법이다(응?). 


### GenericAPIView vs GenericViewSet
`GenericViewSet`은 `GenericAPIView`와 `ViewSet`을 상속받은 클래스이다. 

`generics.GenericAPIView`는 개별적인 API 뷰를 만들 때 사용되며, 모든 세부 사항을 직접 제어할 때 유용합니다. 반면에 `viewsets.GenericViewSet`는 CRUD 작업을 자동화하고 뷰를 라우팅하는 데 유용하며, 일반적으로 라우터와 함께 사용합니다. 선택은 프로젝트의 요구사항과 개발자의 선호에 따라 다를 수 있습니다.

![](https://github.com/aohus/aohus.github.io/blob/main/assets/images/posts/drf-view-class.png)


## references
https://www.cdrf.co/
