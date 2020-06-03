# Darome Django vertimus

Viso Django puslapio kalbą galime pakeisti labai paprastai, tiesiog settings.py faile:

```python
LANGUAGE_CODE = 'lt'
```

Django automatiškai atvaizduoja visą administratoriaus puslapį lietuvių kalba. Tačiau taip neišverčiami mūsų sukurti laukai ir užrašai. Tai padarysime taip:

Iš pradžių į Windows įdiegiame reikiamas programas gettext ir iconv iš čia:
[https://mlocati.github.io/articles/gettext-iconv-windows.html](https://mlocati.github.io/articles/gettext-iconv-windows.html)

Padarykime vertimus puslapyje index.html:
```python
{% extends "base.html" %}
{% load i18n %}
{% block content %}
  <h1>{% trans "Local library" %}</h1>
  <p>{% trans "Welcome to my Library" %}</p>
  <p>{% trans "Today we have:" %}</p>
  <ul>
    <li><strong>{% trans "Books" %}:</strong> {{ num_books }}</li>
    <li><strong>{% trans "Book Instances" %}:</strong> {{ num_instances }}</li>
    <li><strong>{% trans "Now Available" %}:</strong> {{ num_instances_available }}</li>
    <li><strong>{% trans "Authors" %}:</strong> {{ num_authors }}</li>
  </ul>

<p>{% trans "Your visits:" %} {{ num_visits }}</p>
{% endblock %}
```

Pakeičiame modulių laukų pavadinimus:
```python
from django.utils.translation import gettext as _

class Book(models.Model):
    title = models.CharField(_('Title'), max_length=200)
    author = models.ForeignKey('Author', on_delete=models.SET_NULL, null=True, related_name='books')
    summary = models.TextField(_('Summary'), max_length=1000, help_text='Trumpas knygos aprašymas')
    isbn = models.CharField('ISBN', max_length=13,
                            help_text='13 Simbolių <a href="https://www.isbn-international.org/content/what-isbn">ISBN kodas</a>')
    genre = models.ManyToManyField(Genre, help_text='Išrinkite žanrą(us) šiai knygai')
    cover = models.ImageField(_('Cover'), upload_to='covers', null=True)
```

Išverčiame pranešimų žinutes:
```python
@csrf_protect
def register(request):
    if request.method == "POST":
        # pasiimame reikšmes iš registracijos formos
        username = request.POST['username']
        email = request.POST['email']
        password = request.POST['password']
        password2 = request.POST['password2']
        # tikriname, ar sutampa slaptažodžiai
        if password == password2:
            # tikriname, ar neužimtas username
            if User.objects.filter(username=username).exists():
                messages.error(request, _(f'Username {username} already exists!'))
                return redirect('register')
            else:
                # tikriname, ar nėra tokio pat email
                if User.objects.filter(email=email).exists():
                    messages.error(request, _(f'Email {email} already exists!'))
                    return redirect('register')
                else:
                    # jeigu viskas tvarkoje, sukuriame naują vartotoją
                    User.objects.create_user(username=username, email=email, password=password)
                    return redirect('index')
        else:
            messages.error(request, _('Passwords do not match!'))
            return redirect('register')
    return render(request, 'register.html')
```

Dabar mūsų programoje (library) sukuriame katalogą "locale" ir konsolėje paleidžiame šią komandą:
```console
django-admin makemessages -l lt
```
Django automatiškai sukurs vertimų failą .po, kurį galime atsidaryti su [POEdit](https://poedit.net/) programa, pasidaryti vertimus ir juos išsaugoti .mo faile (ten pat). Pabandykime perjungti kalbas ir pažiūrėti, ar pasikeičia kalbos išverstame puslapyje, taip pat išversto modelio admin puslapyje.

Sukompiliuojame vertimus, kad Django galėtų juos naudoti:
```console
python manage.py compilemessages
```

Jei norime, kad kalba būtų automatiškai parenkama pagal naršyklėje nustatytą kalbą, į settings.py pridedame 'django.middleware.locale.LocaleMiddleware':
```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.locale.LocaleMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```

Galime į puslapį įdėti kalbos pasirinkimo formą. Tam:

Į settings.py dedame:
```python
from django.utils.translation import ugettext_lazy as _

LANGUAGES = (
    ('en-us', _('English')),
    ('lt', _('Lithuanian')),
)
```

Į library/urls.py pridedame:
```python
    path(r'^i18n/', include('django.conf.urls.i18n')),
```

Ten, kur norime turėti kalbos pasirinkimo formą, dedame šį kodą (pvz. į base.html meniu):

```html
      <form action="{% url 'set_language' %}" method="post">
        {% csrf_token %}
        <input name="next" type="hidden" value="{{ redirect_to }}"/>
        <select name="language">
          {% get_current_language as LANGUAGE_CODE %}
          {% get_available_languages as LANGUAGES %}
          {% for lang in LANGUAGES %}
          <option value="{{ lang.0 }}" {% if lang.0== LANGUAGE_CODE %} selected="selected" {% endif %}>
            {{ lang.1 }} ({{ lang.0 }})
          </option>
          {% endfor %}
        </select>
        <input type="submit" value="Go"/>
      </form>
```

 ## Užduotis
Tęsti kurti Django užduotį – [Autoservisas](https://github.com/robotautas/kursas/wiki/Django-u%C5%BEduotis:-Autoservisas):
* Visus užrašus, kintamuosius (kiek įmanoma) aprašyti anglų kalba.
* Padaryti vertimus, kad perjungus iš anglų kalbos į lietuvių, automatiškai išsiverstų visi užrašai.
* Padaryti mygtukus (arba pasirinkimą), kuris leistų pasirinkti puslapio kalbą.

[Atsakymas](https://github.com/DonatasNoreika/autoservisas)