# 장고 Forms.py input type 변경

결론은 field 내부의 widget 매개변수를 통해서 type을 스스로 정할 수 있다.

```python
FRUIT_CHOICES= [
    ('orange', 'Orange s'),
    ('cantaloupe', 'Cantaloupes'),
    ('mango', 'Mangoes'),
    ('honeydew', 'Honeydews'),
    ]

class UserForm(forms.Form):
    first_name= forms.CharField(max_length=100)
    last_name= forms.CharField(max_length=100)
    email= forms.EmailField()
    age= forms.IntegerField()
    favorite_fruit= forms.CharField(label='What is your favorite fruit?', widget=forms.RadioSelect(choices=FRUIT_CHOICES))
```

아래 favorite\_fuit변수처럼 forms의 CharField내부의 매개변수로 input type을 변경할 수 있다. 지금 예시는 radio 타입

장고 widget 공식 문서 링크: [https://docs.djangoproject.com/en/3.0/ref/forms/widgets/](https://docs.djangoproject.com/en/3.0/ref/forms/widgets/)

