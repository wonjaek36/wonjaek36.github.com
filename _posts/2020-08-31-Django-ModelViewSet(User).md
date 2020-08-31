---
layout: post
title: "Django ModelViewSet 이용"
date: 2020-08-31 13:23 +0900
lastmod: 2020-08-31 13:23 +0900
tags: [django]
---

### Environment
* Django == 3.0.8
* djangorestframework == 3.11.1



기존 Django 내 User 테이블을 이용하여 User CRUD를 생성해보고자 한다.  
물론, 다른 모델도 ModelViewSet을 적용할 수 있다.

ModelViewSet을 적용하기 위해서는, serializers.py, views.py, urls.py를 수정한다.  

<br />
#### 1. serializers.py
{% highlight python %}
# 기존 User Model import
from django.contrib.auth.models import User

# User Model에 대한 Serializer 정의
# Django의 Model -> Json 으로 바꾸기 위한 작업
class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = '__all__'
{% endhighlight %}

<br />  
#### 2. views.py
{% highlight python %}
# 모든 사용자가 접근할 수 있도록 설정
class UserViewSet(viewsets.ModelViewSet):

    queryset = User.objects.all()
    serializer_class = UserSerializer
{% endhighlight %}

<br />
#### 3. urls.py
{% highlight python %}
user_list = UserViewSet.as_view({
    'get': 'list',
    'post': 'create',
})
user_detail = UserViewSet.as_view({
    'get': 'retrieve',
    'put': 'update',
    'patch': 'partial_update',
    'delete': 'destroy',
})
urlpatterns = [
    ...
    path('user/', user_list, name="user-list"),
    path('user/<int:pk>/', user_detail, name='user-detail'),
    ...
]
{% endhighlight %}

ModelViewSet은 list, create, retrieve, update, partial_update, destroy 함수를 기본적으로 제공하며,
함수명은 CRUD의 Create, Read, Update, Delete와 매칭됩니다.  
user_list와 user_detail은 각각 http method가 어떤 함수와 매칭되는지 알려주는 dict 입니다.  

<br />
### Reference:
* https://www.django-rest-framework.org/api-guide/viewsets/#modelviewset
* https://www.django-rest-framework.org/tutorial/6-viewsets-and-routers/
