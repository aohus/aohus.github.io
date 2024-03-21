---
layout: post
title: "[DRF] Django Rest Framework의 Router"
subtitle:
categories: Django DRF
tags: []
---
  
DRF의 Router 클래스는 뷰셋을 등록하고 해당 뷰셋에 대한 표준 CRUD 작업에 대한 URL 패턴을 자동으로 생성하는 메서드를 제공한다. 이를 통해 개별 뷰셋 액션마다 수동으로 URL 패턴을 정의하는 작업을 추상화한다.   
  
```python  
from rest_framework import routers   
from .views import MyModelViewSet   
  
router = routers.DefaultRouter()   
router.register('mymodels', MyModelViewSet)   
  
urlpatterns = router.urls  
```  
  
drf router 가 해주는 일은 아래와 같다.   
```python  
# urls.py  
  
from django.urls import path from .views import MyModelViewSet  
  
mymodels_viewset = MyModelViewSet.as_view({   
	'get': 'list',   
	'post': 'create',   
})   
mymodel_detail_view = MyModelViewSet.as_view({   
	'get': 'retrieve',   
	'put': 'update',   
	'patch': 'partial_update',   
	'delete': 'destroy',   
})  
  
urlpatterns = [   
	path('mymodels/', mymodels_viewset, name='mymodels-list'),   
	path('mymodels/<int:pk>/', mymodel_detail_view, name='mymodels-detail'),   
]  
```  
  
  
## references  
<https://wikidocs.net/197564>  
  