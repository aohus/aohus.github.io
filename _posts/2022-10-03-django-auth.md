---
layout: post
title: "[Django] Django 인증 시스템(auth)"
subtitle:
categories: Django database
tags: []
---
  
# auth  
  
Django의 사용자 인증 시스템(authentication system)은 authentication : 사용자가 누구인지 확인, authorization : 인증된 사용자가 수행할 수 있는 작업을 결정 을 제공한다.   
  
구체적으로 제공하는 것  
- 사용자 계정   
- Permissions(권한): 사용자가 특정 작업을 수행할 수 있는지 여부  
- Groups(그룹):두 명 이상의 사용자에게 라벨과 권한을 적용하는 일반적인 방법  
- 쿠키 기반 사용자 세션을 처리  
- A configurable password hashing system  
- Forms and view tools for logging in users, or restricting content  
- A pluggable backend system  
  
제공하지 않는 것(해결 위해서는 타사 패키지 사용해야함)  
- 비밀번호 강도 확인  
- 로그인 시도 제한  
- third-parties에 대한 인증 (OAuth, )  
- Object-level permissions  
  
## **로그인 인증 구조**  
---  
 로그인 인증에 사용하는 패키지는 다음과 같다.  
  
```  
django.contrib.auth  
```  
  
 설정 파일 settings.py를 보면, 위 패키지를 사용할 수 있도록 맨 처음에 INSTALLED_APPS에 기재되어 있다.   
  
```  
INSTALLED_APPS = [  
    'django.contrib.admin',  
    'django.contrib.auth',  
    'django.contrib.contenttypes',  
    'django.contrib.sessions',  
    'django.contrib.messages',  
    'django.contrib.staticfiles',  
]  
```  
  
 로그인 인증에 사용하는 모델은 다음의 세 가지이다.  
  
- django.contrib.auth.models.User  
- django.contrib.auth.models.Permission  
- django.contrib.auth.models.Group  
  
 생성한 유저 정보는 위의 User 모델에 저장된다. 「python manage.py createsuperuser」으로 생성된 슈퍼 유저도 User 모델에 저장된다.  
  
  
  
# auth default  
  
## 사용자 객체(User)  
User 객체는 인증 시스템의 핵심이다. 장고에는 하나의 사용자 클래스만 존재한다. 즉 슈퍼유저나 관리자(staff) 유저는 User 객체의 특별한 속성 세트일 뿐이다.   
  
default 유저의 선순위 속성([User model](https://docs.djangoproject.com/en/4.2/ref/contrib/auth/#django.contrib.auth.models.User) full api document)  
- username(required)   
- password(required)   
- email  
- first_name  
- last_name  
- groups - many to many relationship to group  
- user_permissions - many to many relationship to permission  
- is_staff - designates whether this user can access the admin site  
- is_active - 유저 삭제하는 대신, 이 필드의 플래그를 false로 두기를 권장. foriegn key를 지킬 수 있을 것. 유저가 로그인 할 수 있는지와는 상관이 없다.   
- is_superuser - Designates that this user has all permissions without explicitly assigning them.  
- last_login  
- date_joined  
  
### creating users  
```  
>>> from django.contrib.auth.models import User  
>>> user = User.objects.create_user("john", "lennon@thebeatles.com", "johnpassword")  
  
# At this point, user is a User object that has already been saved  
# to the database. You can continue to change its attributes  
# if you want to change other fields.  
>>> user.last_name = "Lennon"  
>>> user.save()  
```  
  
### creating superusers  
```  
$ python manage.py createsuperuser --username=joe --email=joe@example.com  
```  
  
### changing password  
```  
>>> from django.contrib.auth.models import User  
>>> u = User.objects.get(username="john")  
>>> u.set_password("new password")  
>>> u.save()  
```  
  
### Authenticating users  
```  
from django.contrib.auth import authenticate  
  
user = authenticate(username="john", password="secret")  
if user is not None:  
    # A backend authenticated the credentials  
    ...  
else:  
    # No backend authenticated the credentials  
    ...  
```  
  
- 이 방식은 저수준 인증이다. 자체 인증 시스템을 만들면 이렇게 하지 않아도 된다. 유저를 로그인 시키는 방법을 찾고 있다면 [LoginView](https://docs.djangoproject.com/en/4.2/topics/auth/default/#django.contrib.auth.views.LoginView)를 참고  
	-> 근데 이거는 restapi에 쓰는 방법이 아니자나?  
  
## 권한과 인증  
- 유저를 그룹에 추가하거나 그룹에서 제외할 수 있다.   
- view, change, add, delete 등의 permission을 설정할 수 있다.   
```python  
myuser.groups.set([group_list])  
myuser.groups.add(group, group, ...)  
myuser.groups.remove(group, group, ...)  
myuser.groups.clear()  
myuser.user_permissions.set([permission_list])  
myuser.user_permissions.add(permission, permission, ...)  
myuser.user_permissions.remove(permission, permission, ...)  
myuser.user_permissions.clear()  
```  
  
### default permissions  
djangolcontib.auth가 INSTALLED_APPS 에 있으면, 기본 권한(add, change, delete, view)이 installed applications에 있는 각각의 장고 모델에 생성된다. 권한은 manage.py migrate 할 때 생성된다. #q 각각의 모델에 권한이 생성된다는 것의 의미??  
  
권한이 있는지는 아래와 같이 확인할 수 있다.   
```python  
# user가 'cafe app - comment 모델'에 대해 add 권한있는지 확인  
user.has_perm('cafe.add_comment')  
```  
### Groups  
유저를 카테고라이징해서 권한을 나눌 수 있다.   
  
### Programmatically creating permissions  
While [custom permissions](https://docs.djangoproject.com/en/4.2/topics/auth/customizing/#custom-permissions) can be defined within a model’s `Meta` class, you can also create permissions directly. For example, you can create the `can_publish` permission for a `BlogPost` model in `myapp`: #q content_type 뭐지?  
  
```python  
from myapp.models import BlogPost  
from django.contrib.auth.models import Permission  
from django.contrib.contenttypes.models import ContentType  
  
content_type = ContentType.objects.get_for_model(BlogPost)  
permission = Permission.objects.create(  
    codename="can_publish",  
    name="Can Publish Posts",  
    content_type=content_type,  
)  
```  
  
**모델에서..** #q can publish에 대한 정의는?  
```python  
class User(models.Model):  
    class Meta:  
        permissions = [("can_publish", "Can Publish Posts")]  
```  
### 권한 캐싱  
권한은 일반적으로 추가된 후 즉시 확인되지 않는다. 권한을 추가하고 즉시 확인하는 경우 가장 쉬운 해결 방법은 데이터베이스에서 사용자를 다시 가져오는 것이다.  
  
## 웹 요청에서 인증  
장고는 웹 요청에서의 인증 시스템을 위해 sessions과  미들웨어를 사용한다.   
유저가 로그인 했다고 할 때, `request.user`를 사용해서 매 요청에 현재 유저의 정보를 제공한다.   
  
### 로그인  
```python  
from django.contrib.auth import authenticate, login  
  
def my_view(request):  
    username = request.POST["username"]  
    password = request.POST["password"]  
    user = authenticate(request, username=username, password=password)  
    if user is not None:  
        login(request, user)  
        # Redirect to a success page.  
        ...  
    else:  
        # Return an 'invalid login' error message.  
```  
  
login 함수를 사용하면, 장고는 유저 id를 세션 프레임의 세션에에 저장한다.   
#q 어떤 형태로 저장되고 다음엔 어떻게 쓰이는지 알아보자  
  
### 로그아웃  
```python  
from django.contrib.auth import logout  
  
def logout_view(request):  
    logout(request)  
    # Redirect to a success page.  
```  
유저가 로그인 한 적 없더라고 logout 함수는 에러를 반환하지 않는다. logout 하면 session data가 지워진다.   
  
### Limiting access to logged-in users  
#### The `login_required` decorator  
```python  
from django.contrib.auth.decorators import login_required  
  
@login_required(login_url="/accounts/login/")  
def my_view(request):  
    ...  
```  
- If the user isn’t logged in, redirect to [`settings.LOGIN_URL`](https://docs.djangoproject.com/en/4.2/ref/settings/#std-setting-LOGIN_URL)(`login_url= ` 없을 때), passing the current absolute path in the query string. Example: `/accounts/login/?next=/polls/3/`.  
- If the user is logged in, execute the view normally. The view code is free to assume the user is logged in.  
- `@staff_member_required` 도 사용 가능함.  
  
#### The `LoginRequiredMixin` mixin  
class base view 이용하면, 모든 함수에 login_required 데코레이터를 다는 대신, LoginRequiredMixin 상속 받아서 이용할 수 있다. 인증되지 않은 모든 요청은 raise_exception 변수에 의해 login page로 리다이렉션 되거나 403 에러를 반환한다.   
```python  
from django.contrib.auth.mixins import LoginRequiredMixin  
  
class MyView(LoginRequiredMixin, View):  
    login_url = "/login/"  
    redirect_field_name = "redirect_to"  
```  
  
- 인증되지 않은 요청을 커스텀하기 위해서는 [AccessMixin](https://docs.djangoproject.com/en/4.2/topics/auth/default/#django.contrib.auth.mixins.AccessMixin) 을 자세히 보기  
	**raise_exception**  
	- If this attribute is set to `True`, a [`PermissionDenied`](https://docs.djangoproject.com/en/4.2/ref/exceptions/#django.core.exceptions.PermissionDenied "django.core.exceptions.PermissionDenied") exception is raised when the conditions are not met. When `False` (the default), anonymous users are redirected to the login page.  
  
#### The `permission_required` decorator  
```python  
from django.contrib.auth.decorators import login_required, permission_required  
  
@login_required  
@permission_required("polls.add_choice", raise_exception=True)  
def my_view(request):  
    ...  
```  
  
- If the `raise_exception` parameter is given, the decorator will raise `PermissionDenied` prompting the 403(http forbidden) view nstead of redirecting to the login page.  
  
#### The `PermissionRequiredMixin` mixin  
class base view 이용할 때, permission 확인  
```python  
from django.contrib.auth.mixins import PermissionRequiredMixin  
  
class MyView(PermissionRequiredMixin, View):  
    permission_required = "polls.add_choice"  
    # Or multiple of permissions:  
    permission_required = ["polls.view_choice", "polls.change_choice"]  
```  
  
  
# auth customizing  
  
## Other authentication sources  
다른 인증 소스, 즉 사용자 이름과 비밀번호 또는 인증 방법의 다른 소스에 연결해야 하는 경우가 있을 수 있습니다.  
  
pass  
  
## Custom Permission  
```python  
class Task(models.Model):  
    ...  
  
    class Meta:  
        permissions = [  
            ("change_task_status", "Can change the status of tasks"),  
            ("close_task", "Can remove a task by setting its status as closed"),  
        ]  
```  
  
## Extending existing User   
1) [proxy model](https://docs.djangoproject.com/en/4.2/topics/db/models/#proxy-models)  based on User: User 모델의 db 그대로 사용하고, 기타 동작만 수정하여 사용할 때  
2) profile model(use OneToOneField): User 관련 추가 데이터 저장하고 싶을 때  
	- non-auth 정보 저장 가능  
```python  
from django.contrib.auth.models import User  
  
class Employee(models.Model):  
    user = models.OneToOneField(User, on_delete=models.CASCADE)  
    department = models.CharField(max_length=100)  
```  
  
##  사용자 모델(existing User) 대체  
1) AbstractUser  
2) AbstractBaseUser : 인증 시스템을 커스텀해서 구현하고 싶을 때  
### 프로젝트를 시작할 때 맞춤 사용자 모델 사용하기  
  
> 만약 독자가 새로운 프로젝트를 시작한다면 장고의 기본 `User` 모델이 독자에게 충분하더라도 맞춤 사용자 모델을 설정할 것을 강력히 추천합니다.이 모델은 기본 사용자 모델과 똑같이 행동하지만 만약 필요한 일이 생긴다면 이후에 모델을 커스텀할 수 있습니다:  
  
사용자 모델을 사용하기 위해서는 아래와 같이 `AbstractUser` 를 상속받아 User 모델을 만들고  
```python  
from django.contrib.auth.models import AbstractUser  
  
class User(AbstractUser):  
    pass  
```  
  
AUTH_USER_MODEL 을 변경하고  
```  
AUTH_USER_MODEL = "myapp.MyUser"  
```  
  
`admin.py` 에도 모델을 등록해 주어야한다.   
```python  
from django.contrib import admin  
from django.contrib.auth.admin import UserAdmin  
from .models import User  
  
admin.site.register(User, UserAdmin)  
```  
  
#q reusable apps?? https://docs.djangoproject.com/en/4.2/topics/auth/customizing/#reusable-apps-and-auth-user-model  
  
-> 재사용 가능한 앱이 무엇인가? 이 앱을 활용해 뭔가를 하도록 만들어주는 것? 그럼 자체 인증시스템을 쓸 수도 있으니 AbstractUser 모델 사용하지 말고, 더 자유로운 AbstractBaseUser 쓰라는 것인가?   
-> 그럼 나는 상관없지.   
  
### custom user model 지정하기  
모든 사용자 관련 정보를 하나의 모델에 유지하면 관련 모델을 검색하기 위해 추가적이거나 더 복잡한 데이터베이스 쿼리가 필요하지 않다. 반면, 사용자 지정 사용자 모델과 관련된 모델에 앱별 사용자 정보를 저장하는 것이 더 적합할 수 있다. 이를 통해 각 앱은 잠재적으로 다른 앱의 가정과 충돌하거나 위반하지 않고 자체 사용자 데이터 요구 사항을 지정할 수 있다. 이는 또한 사용자 모델을 가능한 한 단순하게 유지하고 인증에 중점을 두고 Django가 사용자 정의 사용자 모델이 충족할 것으로 기대하는 최소 요구 사항을 준수한다는 것을 의미한다.  
  
기본 인증 백엔드를 사용하는 경우 모델에는 식별 목적으로 사용할 수 있는 단일 고유 필드가 있어야 한다. 이는 사용자 이름, 이메일 주소 또는 기타 고유한 속성일 수 있다. 이를 지원할 수 있는 사용자 정의 인증 백엔드를 사용하는 경우 고유하지 않은 사용자 이름 필드가 허용된다.  
  
호환되는 사용자 정의 사용자 모델을 구성하는 가장 쉬운 방법은 AbstractBaseUser에서 상속하는 것이다. AbstractBaseUser는 해시된 비밀번호 및 토큰화된 비밀번호 재설정을 포함하여 사용자 모델의 핵심 구현을 제공한다. 그런 다음 몇 가지 주요 구현 세부정보를 제공해야 한다.  
  