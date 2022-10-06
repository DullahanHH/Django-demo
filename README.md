# Django-demo笔记
文档：https://docs.djangoproject.com/zh-hans/4.1/intro/tutorial01/

Django的安装：
1. 配置虚拟环境，安装Django
2. 激活虚拟环境， 运行 django-admin startproject mysite 来创建一个名为mysite的Django项目
3. 切换到有manage.py的目录，此为根目录，运行 python manage.py runserver 来开启服务器

Django中的App：
1. 一个项目中可以存在多个App
2. 运行 python manage.py startapp polls 在项目中创建一个名为polls的App
3. 在项目（mysite）里的urls.py中的urlpatterns添加 path('polls/', include('polls.urls')),将项目与App建立联系

模型（models）与设置数据库（详见part2）：
1. 为models.py中添加模型或修改模型
2. 在 setting.py 中为 INSTALLED_APPS 添加App Config。（这里的PollsConfig在 polls/apps.py 里，点路径为 polls.apps.PollsConfig）
3. 运行 python manage.py makemigrations 为模型的改变生成迁移文件。
4. 运行 python manage.py migrate 来应用数据库迁移。

python manage.py shell 运行shell
python manage.py createsuperuser 创建管理员

admin.py:
from .models import Question
admin.site.register(Question)
（为管理员界面添加Question模组）
