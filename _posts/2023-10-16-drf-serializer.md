---
layout: post
title: "[DRF] Django Rest Framework의 Serializer 사용법"
subtitle:
categories: Django DRF
tags: []
---

  
## Serializers  
기본 시리얼라이저는 Model에 정의한 코드를 거의 그대로 반복한다.   

```python  
from rest_framework import serializers  
from snippets.models import Snippet, LANGUAGE_CHOICES, STYLE_CHOICES  
  
  
class SnippetSerializer(serializers.Serializer):  
    id = serializers.IntegerField(read_only=True)  
    title = serializers.CharField(required=False, allow_blank=True, max_length=100)  
    code = serializers.CharField(style={'base_template': 'textarea.html'})  
    linenos = serializers.BooleanField(required=False)  
    language = serializers.ChoiceField(choices=LANGUAGE_CHOICES, default='python')  
    style = serializers.ChoiceField(choices=STYLE_CHOICES, default='friendly')  
  
    def create(self, validated_data):  
        """  
        Create and return a new `Snippet` instance, given the validated data.  
        """  
        return Snippet.objects.create(**validated_data)  
  
    def update(self, instance, validated_data):  
        """  
        Update and return an existing `Snippet` instance, given the validated data.  
        """  
        instance.title = validated_data.get('title', instance.title)  
        instance.code = validated_data.get('code', instance.code)  
        instance.linenos = validated_data.get('linenos', instance.linenos)  
        instance.language = validated_data.get('language', instance.language)  
        instance.style = validated_data.get('style', instance.style)  
        instance.save()  
        return instance  
```  
  
## Using ModelSerializers  
```python  
class SnippetSerializer(serializers.ModelSerializer):  
    class Meta:  
        model = Snippet  
        fields = ['id', 'title', 'code', 'linenos', 'language', 'style']  
```  
- An automatically determined set of fields.  
- Simple default implementations for the `create()` and `update()` methods.  
  
  
## Serializer 사용법  
```python  
from snippets.models import Snippet  
from snippets.serializers import SnippetSerializer  
from rest_framework.renderers import JSONRenderer  
from rest_framework.parsers import JSONParser  
  
snippet = Snippet(code='foo = "bar"\n')  
snippet.save()  
  
snippet = Snippet(code='print("hello, world")\n')  
snippet.save()  
```  
  
### Serialization  
```python  
## model object to python native datatype ##   
serializer = SnippetSerializer(snippet)  
serializer.data  
# {'id': 2, 'title': '', 'code': 'print("hello, world")\n', 'linenos': False, 'language': 'python', 'style': 'friendly'}  
  
  
## python native datatype to json ##  
content = JSONRenderer().render(serializer.data)  
content  
# b'{"id": 2, "title": "", "code": "print(\\"hello, world\\")\\n", "linenos": false, "language": "python", "style": "friendly"}'  
```  
  
### Deserialization  
```python  
import io  
  
## json to python native datatype ##  
stream = io.BytesIO(content)  
data = JSONParser().parse(stream)  
  
## python native datatype to model object ##   
serializer = SnippetSerializer(data=data)  
serializer.is_valid()  
# True  
serializer.validated_data  
# OrderedDict([('title', ''), ('code', 'print("hello, world")\n'), ('linenos', False), ('language', 'python'), ('style', 'friendly')])  
serializer.save()  
# <Snippet: Snippet object>  
```  