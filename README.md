# Flask — (boshlang'ichdan o'rtaga)

---

## Mundarija

1. Flask nima va u qanday ishlaydi
2. O'rnatish va loyiha strukturasi
3. Route va View funksiyalar
4. Dinamik URL'lar
5. HTML shablonlar (Jinja2)
6. Shablonda shart va sikllar
7. Template inheritance (asosiy shablon)
8. Static fayllar (CSS, JS, rasm)
9. Ma'lumotlar bazasi — SQLAlchemy sozlash
10. Modellar yaratish
11. Ma'lumot qo'shish
12. Query — ma'lumot o'qish
13. Modellar o'rtasida bog'liqlik (Relationship)
14. Ma'lumot yangilash va o'chirish
15. Blueprint — loyihani bo'limlarga ajratish
16. Xato sahifalari
17. Config — sozlamalar

---

## 1. Flask nima va u qanday ishlaydi

Flask — Python'da yozilgan web framework. Foydalanuvchi brauzerdan so'rov yuboradi, Flask uni qabul qilib, tegishli funksiyani ishga tushiradi va javob qaytaradi.

```
Brauzer  →  /haqida  →  Flask  →  haqida() funksiyasi  →  HTML  →  Brauzer
```

Flask ikki asosiy tushunchaga qurilgan: **route** (qaysi manzilga javob berish) va **view funksiya** (nima qaytarish).

---

## 2. O'rnatish va loyiha strukturasi

```bash
pip install flask flask-sqlalchemy
```

Kichik loyiha uchun tavsiya etiladigan struktura:

```
loyiha/
├── app.py
├── models.py
├── static/
│   └── style.css
└── templates/
    ├── base.html
    ├── index.html
    └── detail.html
```

Katta loyiha uchun blueprint ishlatiladi — bu haqda 15-bo'limda.

---

## 3. Route va View funksiyalar

Route — URL manzilini Python funksiyasiga bog'laydi. `@app.route()` dekorator sifatida yoziladi.

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def bosh():
    return 'Bosh sahifa'

@app.route('/haqida')
def haqida():
    return 'Haqida sahifasi'

@app.route('/kontakt')
def kontakt():
    return 'Kontakt sahifasi'

if __name__ == '__main__':
    app.run(debug=True)
```

Bir funksiyaga bir nechta URL bog'lash ham mumkin:

```python
@app.route('/')
@app.route('/bosh')
def bosh():
    return 'Bosh sahifa'
```

---

## 4. Dinamik URL'lar

URL ichidagi o'zgaruvchi qism `<qavs>` ichida yoziladi va funksiyaga argument sifatida keladi.

```python
@app.route('/foydalanuvchi/<ism>')
def foydalanuvchi(ism):
    return f'Salom, {ism}!'

# /foydalanuvchi/Ali  →  Salom, Ali!
# /foydalanuvchi/Vali →  Salom, Vali!
```

Tur belgilash — faqat raqam qabul qilish:

```python
@app.route('/mahsulot/<int:id>')
def mahsulot(id):
    return f'Mahsulot ID: {id}'

# /mahsulot/5   →  Mahsulot ID: 5
# /mahsulot/abc →  404 xato (raqam emas)
```

Turlar:

| Tur | Ma'nosi |
|---|---|
| `<id>` | Istalgan matn |
| `<int:id>` | Butun son |
| `<float:narx>` | O'nli son |

---

## 5. HTML shablonlar (Jinja2)

To'g'ridan-to'g'ri HTML yozish o'rniga `templates/` papkasidagi fayllar ishlatiladi. `render_template()` funksiyasi shablonni topib, o'zgaruvchilarni joylashtiradi.

`templates/index.html`:
```html
<!DOCTYPE html>
<html>
  <head><title>Bosh sahifa</title></head>
  <body>
    <h1>Salom, {{ ism }}!</h1>
    <p>Yosh: {{ yosh }}</p>
  </body>
</html>
```

`app.py`:
```python
from flask import Flask, render_template

app = Flask(__name__)

@app.route('/')
def bosh():
    return render_template('index.html', ism='Ali', yosh=25)
```

`{{ o'zgaruvchi }}` — qiymatni sahifaga chiqaradi. Istalcha ko'p o'zgaruvchi uzatish mumkin.

---

## 6. Shablonda shart va sikllar

Jinja2 shablonida Python'ga o'xshash shart va sikllar yoziladi, lekin `{% %}` qavslari ichida.

### Shart

```html
{% if user.yosh >= 18 %}
  <p>Siz voyaga yetgansiz.</p>
{% elif user.yosh >= 13 %}
  <p>Siz o'smírsiz.</p>
{% else %}
  <p>Siz bolasiz.</p>
{% endif %}
```

### Sikl

```html
<ul>
  {% for user in foydalanuvchilar %}
    <li>{{ user.ism }} — {{ user.yosh }} yosh</li>
  {% endfor %}
</ul>
```

### Sikl ichida maxsus o'zgaruvchilar

```html
{% for user in foydalanuvchilar %}
  <p>{{ loop.index }}. {{ user.ism }}</p>   <!-- 1, 2, 3... -->
{% endfor %}
```

### Bo'sh ro'yxat holati

```html
{% for user in foydalanuvchilar %}
  <li>{{ user.ism }}</li>
{% else %}
  <p>Hozircha hech kim yo'q.</p>
{% endfor %}
```

---

## 7. Template inheritance — asosiy shablon

Har bir sahifaga `<head>`, navbar, footer qayta yozmaslik uchun bitta asosiy shablon yaratiladi. Boshqa shablonlar undan meros oladi.

`templates/base.html`:
```html
<!DOCTYPE html>
<html>
  <head>
    <title>{% block title %}Saytim{% endblock %}</title>
    <link rel="stylesheet" href="/static/style.css">
  </head>
  <body>

    <nav>
      <a href="/">Bosh sahifa</a>
      <a href="/haqida">Haqida</a>
    </nav>

    <main>
      {% block content %}{% endblock %}
    </main>

    <footer>
      <p>© 2024 Mening saytim</p>
    </footer>

  </body>
</html>
```

`templates/index.html`:
```html
{% extends 'base.html' %}

{% block title %}Bosh sahifa{% endblock %}

{% block content %}
  <h1>Xush kelibsiz!</h1>
  <p>Bu bosh sahifa.</p>
{% endblock %}
```

`templates/haqida.html`:
```html
{% extends 'base.html' %}

{% block title %}Haqida{% endblock %}

{% block content %}
  <h1>Biz haqimizda</h1>
  <p>Bu haqida sahifasi.</p>
{% endblock %}
```

Barcha sahifalar nav va footer ni avtomatik oladi.

---

## 8. Static fayllar — CSS, JS, rasm

`static/` papkasidagi fayllar to'g'ridan-to'g'ri brauzerga uzatiladi. `url_for()` funksiyasi to'g'ri yo'lni topadi.

`static/style.css`:
```css
body {
    font-family: sans-serif;
    max-width: 800px;
    margin: 0 auto;
    padding: 20px;
}

nav a {
    margin-right: 12px;
    text-decoration: none;
    color: #333;
}
```

`templates/base.html` ichida:
```html
<link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
<img src="{{ url_for('static', filename='rasm.png') }}">
<script src="{{ url_for('static', filename='main.js') }}"></script>
```

`url_for()` ni ishlatish tavsiya etiladi — yo'lni qo'lda yozishdan xatodan saqlaydi.

---

## 9. Ma'lumotlar bazasi — SQLAlchemy sozlash

SQLAlchemy — Python klasslari orqali SQL so'rovlarini avtomatik yozib beruvchi ORM (Object Relational Mapper). SQL bilmasdan ham baza bilan ishlash mumkin.

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)

app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///baza.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db = SQLAlchemy(app)
```

`sqlite:///baza.db` — loyiha papkasida `baza.db` fayli yaratiladi. Alohida server kerak emas, boshlash uchun ideal.

Keyinchalik PostgreSQL yoki MySQL ga o'tish uchun faqat shu qatorni o'zgartirish kifoya:

```python
# PostgreSQL
app.config['SQLALCHEMY_DATABASE_URI'] = 'postgresql://user:parol@localhost/bazanomi'

# MySQL
app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql://user:parol@localhost/bazanomi'
```

---

## 10. Modellar yaratish

Har bir model — bazadagi bir jadval. `db.Model` dan meros olinadi, ustunlar `db.Column` bilan belgilanadi.

```python
class Foydalanuvchi(db.Model):
    id       = db.Column(db.Integer, primary_key=True)
    ism      = db.Column(db.String(100), nullable=False)
    email    = db.Column(db.String(120), unique=True, nullable=False)
    yosh     = db.Column(db.Integer)
    faol     = db.Column(db.Boolean, default=True)
    yaratildi = db.Column(db.DateTime, default=db.func.now())

    def __repr__(self):
        return f'<Foydalanuvchi {self.ism}>'
```

Ustun turlari:

| Tur | Ma'nosi |
|---|---|
| `db.Integer` | Butun son |
| `db.String(n)` | Matn, maksimal n belgi |
| `db.Text` | Uzun matn (cheksiz) |
| `db.Boolean` | True / False |
| `db.Float` | O'nli son |
| `db.DateTime` | Sana va vaqt |

Ustun parametrlari:

| Parametr | Ma'nosi |
|---|---|
| `primary_key=True` | Unikal kalit, avtomatik raqamlanadi |
| `nullable=False` | Bo'sh bo'lmasligi shart |
| `unique=True` | Takrorlanmasligi shart |
| `default=...` | Standart qiymat |

Jadval yaratish — dastur birinchi marta ishga tushganda yoki alohida skript bilan:

```python
with app.app_context():
    db.create_all()
```

---

## 11. Ma'lumot qo'shish

```python
with app.app_context():
    # bitta yozuv
    user = Foydalanuvchi(ism='Ali', email='ali@mail.com', yosh=25)
    db.session.add(user)
    db.session.commit()

    # bir vaqtda bir nechta yozuv
    users = [
        Foydalanuvchi(ism='Vali', email='vali@mail.com', yosh=30),
        Foydalanuvchi(ism='Guli', email='guli@mail.com', yosh=22),
        Foydalanuvchi(ism='Soli', email='soli@mail.com', yosh=28),
    ]
    db.session.add_all(users)
    db.session.commit()
```

`db.session` — bazaga yozishdan oldingi vaqtinchalik maydon. `commit()` chaqirilmaguncha haqiqatda saqlanmaydi.

---

## 12. Query — ma'lumot o'qish

### Hammasini olish

```python
users = Foydalanuvchi.query.all()
# [<Foydalanuvchi Ali>, <Foydalanuvchi Vali>, ...]

for user in users:
    print(user.ism, user.yosh)
```

### Birinchisini olish

```python
user = Foydalanuvchi.query.first()
print(user.ism)  # Ali
```

### ID bo'yicha olish

```python
user = Foydalanuvchi.query.get(1)   # ID = 1
print(user.ism)   # Ali

user = Foydalanuvchi.query.get(99)  # topilmasa
print(user)       # None
```

### Oddiy shart — `filter_by()`

```python
# email bo'yicha
user = Foydalanuvchi.query.filter_by(email='ali@mail.com').first()

# yosh bo'yicha barcha mos keladiganlar
users = Foydalanuvchi.query.filter_by(yosh=25).all()

# bir vaqtda ikki shart
users = Foydalanuvchi.query.filter_by(yosh=25, faol=True).all()
```

### Murakkab shart — `filter()`

```python
# 25 yoshdan katta
users = Foydalanuvchi.query.filter(Foydalanuvchi.yosh > 25).all()

# 20 va 30 oralig'ida
users = Foydalanuvchi.query.filter(
    Foydalanuvchi.yosh >= 20,
    Foydalanuvchi.yosh <= 30
).all()

# ismida "ali" borlar (katta-kichik harfga sezgir emas)
users = Foydalanuvchi.query.filter(
    Foydalanuvchi.ism.ilike('%ali%')
).all()
```

### OR — yoki sharti

```python
from sqlalchemy import or_

users = Foydalanuvchi.query.filter(
    or_(Foydalanuvchi.ism == 'Ali', Foydalanuvchi.ism == 'Vali')
).all()
```

### Tartiblash

```python
from sqlalchemy import desc

# yoshdan kichikdan kattaga (default)
users = Foydalanuvchi.query.order_by(Foydalanuvchi.yosh).all()

# yoshdan kattadan kichikka
users = Foydalanuvchi.query.order_by(desc(Foydalanuvchi.yosh)).all()
```

### Cheklash va sahifalash

```python
# faqat 5 ta
users = Foydalanuvchi.query.limit(5).all()

# 5 tasini o'tkazib, keyingi 5 tasini olish (2-sahifa)
users = Foydalanuvchi.query.offset(5).limit(5).all()
```

### Sanoq

```python
jami = Foydalanuvchi.query.count()           # hammasi
faollar = Foydalanuvchi.query.filter_by(faol=True).count()
```

### Zanjirlab yozish

Barcha metodlar zanjir shaklida birlashtirilishi mumkin:

```python
users = (Foydalanuvchi.query
         .filter(Foydalanuvchi.faol == True)
         .order_by(Foydalanuvchi.ism)
         .limit(10)
         .all())
```

---

## 13. Modellar o'rtasida bog'liqlik — Relationship

Haqiqiy loyihalarda modellar bir-biriga bog'liq bo'ladi. Masalan, bir foydalanuvchining ko'p postlari bo'lishi mumkin.

### Bir-ko'p (One-to-Many)

```python
class Foydalanuvchi(db.Model):
    id     = db.Column(db.Integer, primary_key=True)
    ism    = db.Column(db.String(100), nullable=False)
    postlar = db.relationship('Post', backref='muallif', lazy=True)
    # backref — Post modelida .muallif degan attribute paydo bo'ladi

class Post(db.Model):
    id              = db.Column(db.Integer, primary_key=True)
    sarlavha        = db.Column(db.String(200), nullable=False)
    matn            = db.Column(db.Text)
    foydalanuvchi_id = db.Column(db.Integer, db.ForeignKey('foydalanuvchi.id'))
```

Ishlatish:

```python
# foydalanuvchining barcha postlari
user = Foydalanuvchi.query.get(1)
print(user.postlar)        # [<Post ...>, <Post ...>]

# postning muallifi
post = Post.query.get(1)
print(post.muallif.ism)    # Ali
```

Post qo'shish:

```python
user = Foydalanuvchi.query.get(1)

post = Post(sarlavha='Birinchi post', matn='Mazmun...', muallif=user)
db.session.add(post)
db.session.commit()
```

### Ko'p-ko'p (Many-to-Many)

Masalan, bir post ko'p tegga ega bo'lishi, bir teg ko'p postda bo'lishi mumkin. Buning uchun oraliq jadval kerak.

```python
# oraliq jadval (klass emas, oddiy jadval)
post_teg = db.Table('post_teg',
    db.Column('post_id', db.Integer, db.ForeignKey('post.id')),
    db.Column('teg_id',  db.Integer, db.ForeignKey('teg.id'))
)

class Post(db.Model):
    id       = db.Column(db.Integer, primary_key=True)
    sarlavha = db.Column(db.String(200))
    teglar   = db.relationship('Teg', secondary=post_teg, backref='postlar')

class Teg(db.Model):
    id   = db.Column(db.Integer, primary_key=True)
    nomi = db.Column(db.String(50))
```

Ishlatish:

```python
post = Post.query.get(1)
teg  = Teg.query.filter_by(nomi='python').first()

post.teglar.append(teg)   # tegni postga qo'shish
db.session.commit()

print(post.teglar)         # [<Teg python>]
print(teg.postlar)         # [<Post Birinchi post>]
```

### lazy parametri

```python
db.relationship('Post', backref='muallif', lazy=True)
```

| Qiymat | Ma'nosi |
|---|---|
| `lazy=True` | So'ralganda yuklanadi (tavsiya etiladi) |
| `lazy='joined'` | Asosiy query bilan birga yuklanadi |
| `lazy='dynamic'` | Query obyekti qaytaradi, keyin filter qilish mumkin |

---

## 14. Ma'lumot yangilash va o'chirish

### Yangilash

```python
user = Foydalanuvchi.query.get(1)
user.ism  = 'Alibek'      # qiymatlar o'zgartirildi
user.yosh = 26
db.session.commit()        # saqlanadi
```

### Bir vaqtda ko'pini yangilash

```python
Foydalanuvchi.query.filter_by(faol=False).update({'yosh': 0})
db.session.commit()
```

### O'chirish

```python
user = Foydalanuvchi.query.get(1)
db.session.delete(user)
db.session.commit()
```

### Bir vaqtda ko'pini o'chirish

```python
Foydalanuvchi.query.filter(Foydalanuvchi.yosh < 18).delete()
db.session.commit()
```

---

## 15. Blueprint — loyihani bo'limlarga ajratish

Loyiha kattalashganda barcha route'lar bitta `app.py` da bo'lishi noqulay. Blueprint har bir bo'limni alohida faylga ajratadi.

Struktura:

```
loyiha/
├── app.py
├── models.py
├── users/
│   ├── __init__.py
│   └── routes.py
└── posts/
    ├── __init__.py
    └── routes.py
```

`users/routes.py`:
```python
from flask import Blueprint, render_template
from models import Foydalanuvchi

users = Blueprint('users', __name__)

@users.route('/users')
def royhati():
    barcha = Foydalanuvchi.query.all()
    return render_template('users/royhat.html', users=barcha)

@users.route('/users/<int:id>')
def detail(id):
    user = Foydalanuvchi.query.get(id)
    return render_template('users/detail.html', user=user)
```

`posts/routes.py`:
```python
from flask import Blueprint, render_template
from models import Post

posts = Blueprint('posts', __name__)

@posts.route('/posts')
def royhati():
    barcha = Post.query.all()
    return render_template('posts/royhat.html', posts=barcha)
```

`app.py`:
```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///baza.db'
db = SQLAlchemy(app)

from users.routes import users
from posts.routes import posts

app.register_blueprint(users)
app.register_blueprint(posts)

if __name__ == '__main__':
    app.run(debug=True)
```

---

## 16. Xato sahifalari

Standart xato sahifalari o'rniga o'zingiznikini qo'yish:

```python
@app.errorhandler(404)
def topilmadi(e):
    return render_template('404.html'), 404

@app.errorhandler(500)
def server_xato(e):
    return render_template('500.html'), 500
```

`templates/404.html`:
```html
{% extends 'base.html' %}

{% block content %}
  <h1>404 — Sahifa topilmadi</h1>
  <p>Siz izlagan sahifa mavjud emas.</p>
  <a href="/">Bosh sahifaga qaytish</a>
{% endblock %}
```

---

## 17. Config — sozlamalar

Sozlamalarni alohida faylga chiqarish — katta loyihalarda qulay va xavfsiz.

`config.py`:
```python
class Config:
    SECRET_KEY = 'maxfiy-kalit-bu-yerda'
    SQLALCHEMY_DATABASE_URI = 'sqlite:///baza.db'
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    DEBUG = False

class DevConfig(Config):
    DEBUG = True

class ProdConfig(Config):
    SQLALCHEMY_DATABASE_URI = 'postgresql://user:parol@localhost/baza'
```

`app.py`:
```python
from config import DevConfig

app = Flask(__name__)
app.config.from_object(DevConfig)
```

---

## Xulosa — o'rganilgan tushunchalar

| Bo'lim | Tushuncha |
|---|---|
| Route | URL → funksiya bog'liqlig'i |
| Dinamik URL | `<int:id>` bilan o'zgaruvchi manzillar |
| Jinja2 | `{{ }}` va `{% %}` bilan shablonlar |
| Template inheritance | `base.html` dan meros olish |
| Static fayllar | CSS/JS/rasmlarni `static/` da saqlash |
| SQLAlchemy | ORM orqali SQL'siz baza ishlash |
| Model | Jadval = klass, ustun = attribute |
| Query | `.all()`, `.get()`, `.filter()` va boshqalar |
| Relationship | Modellar orasida `ForeignKey` va `relationship` |
| Blueprint | Loyihani bo'limlarga ajratish |
| Config | Sozlamalarni alohida faylda saqlash |

---
