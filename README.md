# Hendrik's Blog – Technische Dokumentation

Ein persönlicher Blog, entwickelt mit Django und deployt auf PythonAnywhere.  
Live: [https://thondorin.pythonanywhere.com](https://thondorin.pythonanywhere.com)

---

## Projektübersicht

Hendrik's Blog ist eine vollständige Web-Applikation auf Basis des Django-Frameworks. Das Projekt umfasst ein eigenes Datenbankmodell, dynamische Views, ein URL-Routing-System sowie ein selbst gestaltetes Frontend ohne externe CSS-Frameworks. Die Applikation unterscheidet zwischen öffentlichen Besuchern und authentifizierten Nutzern.

---

## Technologie-Stack

| Bereich | Technologie |
|---|---|
| Backend | Python 6.0.4, Django 6.0.4 |
| Datenbank | SQLite |
| Frontend | HTML, CSS (custom), Django Template Language |
| Versionskontrolle | Git, GitHub |
| Deployment | PythonAnywhere |

---

## Architektur

Das Projekt folgt dem **MVT-Muster** (Model – View – Template), das Django-eigene Pendant zu MVC:

```
Browser → URL-Router → View → Model (Datenbank) → Template → Browser
```

- **Model** definiert die Datenbankstruktur
- **View** enthält die Geschäftslogik und verarbeitet Anfragen
- **Template** rendert das HTML für den Browser

---

## Datenbankmodell

```python
# blog/models.py

class Post(models.Model):
    author = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    title = models.CharField(max_length=200)
    text = models.TextField()
    created_date = models.DateTimeField(default=timezone.now)
    published_date = models.DateTimeField(blank=True, null=True)

    def publish(self):
        self.published_date = timezone.now()
        self.save()
```

Ein `Post` gehört immer einem `User` (Fremdschlüssel). Nur Posts mit gesetztem `published_date` werden auf der Startseite angezeigt. Nicht veröffentlichte Entwürfe bleiben unsichtbar.

---

## URL-Routing

```python
# blog/urls.py

urlpatterns = [
    path('', views.post_list, name='post_list'),
    path('post/<int:pk>/', views.post_detail, name='post_detail'),
    path('post/new/', views.post_new, name='post_new'),
]
```

| URL | Beschreibung |
|---|---|
| `/` | Startseite mit allen veröffentlichten Posts |
| `/post/1/` | Detailansicht eines einzelnen Posts (per ID) |
| `/post/new/` | Formular zum Erstellen eines neuen Posts |
| `/admin/` | Django Admin Panel (nur Superuser) |

---

## Views

### post_list
Lädt alle veröffentlichten Posts aus der Datenbank, sortiert nach Datum, und übergibt sie dem Template.

```python
def post_list(request):
    posts = Post.objects.filter(
        published_date__lte=timezone.now()
    ).order_by('published_date')
    return render(request, 'blog/post_list.html', {'posts': posts})
```

### post_detail
Lädt einen einzelnen Post anhand seiner ID. Gibt einen 404-Fehler zurück wenn der Post nicht existiert.

```python
def post_detail(request, pk):
    post = get_object_or_404(Post, pk=pk)
    return render(request, 'blog/post_detail.html', {'post': post})
```

### post_new
Verarbeitet das Formular zum Erstellen eines neuen Posts. Nur für eingeloggte Nutzer zugänglich (`@login_required`). Bei gültigem Formular wird der Post gespeichert und der Nutzer zur Detailansicht weitergeleitet.

```python
@login_required
def post_new(request):
    if request.method == "POST":
        form = PostForm(request.POST)
        if form.is_valid():
            post = form.save(commit=False)
            post.author = request.user
            post.published_date = timezone.now()
            post.save()
            return redirect('post_detail', pk=post.pk)
    else:
        form = PostForm()
    return render(request, 'blog/post_edit.html', {'form': form})
```

---

## Formular

```python
# blog/forms.py

class PostForm(forms.ModelForm):
    class Meta:
        model = Post
        fields = ('title', 'text',)
```

Django generiert das Formular automatisch aus dem Model. Die Validierung übernimmt Django ebenfalls – leere Felder werden abgefangen bevor sie in die Datenbank gelangen.

---

## Templates

| Template | Beschreibung |
|---|---|
| `post_list.html` | Startseite mit allen Posts als Cards |
| `post_detail.html` | Vollständige Ansicht eines einzelnen Posts |
| `post_edit.html` | Formular zum Erstellen eines neuen Posts |

Die Templates nutzen die **Django Template Language** für dynamische Inhalte:

```django-html
{% for post in posts %}
    <h2>{{ post.title }}</h2>
    <p>{{ post.text|truncatewords:30 }}</p>
    <a href="{% url 'post_detail' pk=post.pk %}">Read →</a>
{% endfor %}
```

---

## Zugriffssteuerung

Die Applikation unterscheidet zwischen zwei Nutzergruppen:

| Aktion | Öffentlich | Eingeloggt |
|---|---|---|
| Posts lesen | ✅ | ✅ |
| Neuen Post erstellen | ❌ | ✅ |
| Admin-Panel | ❌ | ✅ (Superuser) |

Der `@login_required` Decorator leitet nicht authentifizierte Nutzer automatisch zur Login-Seite weiter, wenn sie versuchen `/post/new/` aufzurufen. Der "New Post" Button ist im Frontend nur für eingeloggte Nutzer sichtbar (`{% if user.is_authenticated %}`).

---

## Deployment

Das Projekt ist auf **PythonAnywhere** deployt. Der Deployment-Prozess läuft über Git:

1. Änderungen lokal committen und auf GitHub pushen
2. Auf PythonAnywhere `git pull origin main` ausführen
3. Web App neu laden über den Reload-Button

PythonAnywhere übernimmt das Hosting, die WSGI-Konfiguration und die statischen Dateien.

---

## Autor

**Thondorin**
