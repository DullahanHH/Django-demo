# Django-demo study note
Doc：https://docs.djangoproject.com/zh-hans/4.1/intro/tutorial01/

Django install：
1. Set up the virtual environment and install Django.
2. Activate the virtual env, run [django-admin startproject mysite] to create a project named 'mysite' in current directory.
3. Move to the directory that has 'manage.py', run [python manage.py runserver] to start the server.


Apps in Django：
1. A Django project can have multiple apps.
2. Run [python manage.py startapp polls] to create an app named 'polls'.
3. 'polls/view.py' view contain what user can see on the web page, for example, a view named 'index' is created.
4. To call the view, we need to map its url.
5. Add [path('', views.index, name='index')] to 'urlpatterns' within 'polls/urls.py'
6. Add [path('polls/', include('polls.urls'))] to 'urlpatterns' within 'mysite/urls.py' to establish the connection between project and app.


Models and Database (Part 2 contents)：
1. In 'models.py', we can add or modify models.
2. To activate models, in 'setting.py' add config path to 'INSTALLED_APPS'.（PollsConfig is inside 'polls/apps.py'，path is 'polls.apps.PollsConfig'）
3. run [python manage.py makemigrations polls] to tell Django you make some changes in models. (Like git commit)
4. run [python manage.py migrate] to create the model table to DB.


Django API:
1. [python manage.py shell] run shell
2. First, we need tp import the model by [from polls.models import Choice, Question]
3. [Question.objects.all()] show all objects in Question
4. [q = Question(question_text="Hello", pub_date=timezone.now())]
5. [q.save()] step 4 and 5 create a Question object and save it to the DB.
6. [q.id] show object's id
7. [q.question_text] show corresponding value
8. If [Question.objects.all()] get a result like '<QuerySet [<Question: Question object (1)>]>', we need to editing in 'polls/models.py' like below.
9. For each model class, add
   def __str__(self):
      return self.question_text
10. The 'question_text' in 'self.question_text' can refer to any value in the model you wants to display.


Django Admin:
1. [python manage.py createsuperuser] create admin.
2. In 'admin.py', add [admin.site.register(Question)] to add Question model to admin site.


View's Detail (Part 3 contents):
1. Multiple views is necessary for an app: 'polls/index', 'polls/detail', 'polls/results', etc.
   Just like the index view, new view should add url to polls.urls
      urlpatterns = [
           path('', views.index, name='index'),
           path('<int:question_id>/', views.detail, name='detail'),
           path('<int:question_id>/results/', views.results, name='results'),
           path('<int:question_id>/vote/', views.vote, name='vote'),
      ]
2. For example, if detail in 'polls.py' is 
   def detail(request, question_id):
       return HttpResponse("You're looking at question %s." % question_id)
3. If someone requests 'polls/34/', it matches detail() view and call
   detail(request=<HttpRequest object>, question_id=34)
4. *NOTES: The 'question_id=34' part comes from <int:question_id>
5. The angle brackets capture the argument and pass to view.


Templates in Django:
1. The templates defined how Django will load and render the page.
2. Under polls directory, create 'templates/polls/index.html'
3. In index.html, [<li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>] refer to the detail view
4. To remove the hardcode by {% url %}: [<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>]
5. It will find the url named as 'detail'.
6. But what if there is another url also named 'detail' but in different app?
7. Therefore, we need to add a namespace for the app.
8. In 'polls/urls.py', add [app_name = 'polls'], and the url in index template should change to:
9. [<li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>]


Post form in Django (Part 4 contents):
1. In 'polls/detail.html', we need to create a post method to let the user vote:
2. [<form action="{% url 'polls:vote' question.id %}" method="post">]
3. Since it is a post process, we need to crypto our info, Django already provide a tag [{% csrf_token %} ], add it to the post form.


Generic views:
1. To reduce the redundant and code less, generic view is a good option.
2. The generic views already provide common patterns, all need to do is to override.
3. First, switch all '<int: question_id>' to '<int: pk>' in 'polls/urls.py'.
4. Then, in 'polls/views.py', we got two generic views, ListView and DetailView.
5. The ResultsView used the generic DetailView:
   class ResultsView(generic.DetailView):
       model = Question
       template_name = 'polls/results.html'
6. The generic DetailView expect capture primary key in url, that's why it changed to 'pk'.
7. use model = 'XXX' to point out which model the view is using.
8. Edit the 'template_name' to the corresponding template.
9. For DetailView the question variable is provided automatically – since we’re using a Django model (model = Question)
10. For ListView, the automatically generated context variable is 'question_list'
11. We need to override 'context_object_name' to 'latest_question_list'
    class IndexView(generic.ListView):
       template_name = 'polls/index.html'
       context_object_name = 'latest_question_list'

       def get_queryset(self):
           return Question.objects.order_by('-pub_date')[:5]


Testing (Part 5 contents):
1. Create test case in 'polls/tests.py'.
2. [self.assertIs(testCase,boolean)] assert the testCast = boolean, if not, test failed.
3. Run test by [python manage.py test polls]


Static styling (Part 6 contents):
1. Create 'polls/static/polls/style.css'. (Same like templates, the structure makes easier when dealing with multiple apps, AppDirectoriesFinder will find 'polls/style.css')
2. When finish coding style.css, go index.html template and insert:
   {% load static %}
   <link rel="stylesheet" href="{% static 'polls/style.css' %}">
3. This will apply the stylesheet to index.html
4. Restart the server can see the style loaded.
5. To add a background image, create 'polls/static/polls/images/' directory, and put the image inside.
6. In stylesheet, do like below:
   body {
      background: white url("images/background.png") no-repeat;
   }


Advance admin site ((Part 7 contents)):
1. Highly customization admin site might be helpful to manipulate data.
2. In 'polls/admin', replace [admin.site.register(Question)] with:
   class QuestionAdmin(admin.ModelAdmin):
      fields = ['pub_date', 'question_text']

   admin.site.register(Question, QuestionAdmin)
3. The above changes make the publishing date come before question text.
   class QuestionAdmin(admin.ModelAdmin):
       fieldsets = [
           (None,               {'fields': ['question_text']}),
           ('Date information', {'fields': ['pub_date']}),
       ]
4. The above changes provide a title for each field. (First title is None since it shows tha site title)