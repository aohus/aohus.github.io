---
layout: post
title: "[DRF] DRF 권한 관리"
subtitle:
categories: Django DRF
tags: []
---
  
## custom permission(object level)  
```python  
from rest_framework import permissions  
  
  
class IsOwnerOrReadOnly(permissions.BasePermission):  
    """  
    Custom permission to only allow owners of an object to edit it.  
    """  
  
    def has_object_permission(self, request, view, obj):  
        # Read permissions are allowed to any request,  
        # so we'll always allow GET, HEAD or OPTIONS requests.  
        if request.method in permissions.SAFE_METHODS:  
            return True  
  
        # Write permissions are only allowed to the owner of the snippet.  
        return obj.owner == request.user  
```  
  
```python  
class CafeListView(viewsets.ModelViewSet):  
	permission_classes = [permissions.IsAuthenticatedOrReadOnly,  
                      IsOwnerOrReadOnly]  
```  
  
  
## 같은 class 안에 메서드 별로 다른 권한 적용  
permissions.py  
```python  
from rest_framework import permissions  
  
class ReadOnlyPermission(permissions.BasePermission):  
    def has_permission(self, request, view):  
        return request.method in permissions.SAFE_METHODS  
  
class CreatePermission(permissions.BasePermission):  
    def has_permission(self, request, view):  
        return request.method == 'POST'  
  
class DeletePermission(permissions.BasePermission):  
    def has_permission(self, request, view):  
        return request.method == 'DELETE'  
```  
  
views.py  
```python  
from rest_framework import viewsets  
from .models import YourModel  
from .permissions import ReadOnlyPermission, CreatePermission, DeletePermission  
  
class YourModelViewSet(viewsets.ModelViewSet):  
    queryset = YourModel.objects.all()  
    serializer_class = YourModelSerializer  
  
    def get_permissions(self):  
        if self.action == 'list':  
            permission_classes = [ReadOnlyPermission]  
        elif self.action == 'create':  
            permission_classes = [CreatePermission]  
        elif self.action == 'destroy':  
            permission_classes = [DeletePermission]  
        else:  
            permission_classes = [permissions.IsAuthenticated]  # 기본 퍼미션  
  
        return [permission() for permission in permission_classes]  
  
```  