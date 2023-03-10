# service_for_courier

## Основное задание

Служба курьерской доставки. Сиситема хранит польователей, пул курьеров и их статусы. В личном кабиненте курьера
отображаются его данные, выполненные заказы, а также текущее задание. В пользовательском приложении есть возможность
выбрать пробукты из списка товаров, а также кнопка оформления заказа и вызова курьера, отображается статус доставки

## Выбор технологии

### Среда

+ для написания кода использовалась кроссплатформенная интегрированная среда разработки PyCharm

### Языки

+ В ходе разработки использовались такие языка программирования как: Python, HTML, CSS, JavaScript

### Фреймворки

+ Для написания backend части использовался фреймворк Django
+ Для создания frontend части использовался фреймворк Bootstrap

### Библиотеки

+ django-phonenumber-field==7.0.2
+ Pillow==9.4.0

## Ход работы

### 1. Разработка пользовательского интерфейса
![index1](https://user-images.githubusercontent.com/98755619/213801188-a359bfe3-952e-4ed0-a152-dea6d9da98ec.png)
![index2](https://user-images.githubusercontent.com/98755619/213807038-3e6078a4-d3d4-438d-a2d3-f340d0e97f44.png)
![заявка на курьера](https://user-images.githubusercontent.com/98755619/213801228-4eca6df1-bde8-455c-b878-3a8dbaa69670.png)
![продукты](https://user-images.githubusercontent.com/98755619/213801251-a56bcd92-8d0a-4741-bc10-d8de4b2b99c6.png)
![корзина](https://user-images.githubusercontent.com/98755619/213801271-c8b1c1ac-f33c-48f9-b5e9-cc7e10832d68.png)

### 2. Описание пользовательских сценариев работы

### Сценарий оформления доставки клиентом

Пользователь заходит на сайт и попадает на главную страничку. Проходит регистрацию и получает доступ к выбору продуктов.
Клиент Добавляет необходимые продукты в карзину и нажимает кнопку "Оформить заказ", далее вводит необходимый данные и
нажимает на "кнопку отправить". У пользователя появляется данный заказ в личном кабинете и по умолчанию статус
выставляется "Ожидает подтверждения". После того как один из курьеров примет заказ в статусе отобразиться "Собирается".
И после того как пользователь получит заказ отобразится "Получен".

### Сценарий регистрации на сайте как курьера

После того как пользователь зарегистрировался на сайте он может оставить заявку на то, что бы стать курьером для этого
надо пролистать в конец сайта и заполнить форму на становление курьером. Далее пользователю позвонит менеджер и при
успехе вышлет на почту ссылку для регистрации курьером. После прохождения данной форму пользователю выдаются права
курьера.

### Сценарий принятия заказа курьером

После того как пользователь получил права курьера он может принимать заказы, которые сейчас доступны, для этого надо
перейти на страничку "биржа заказов". На ней будут отображаться все заказы, которые ожидают подтверждения. Если курьер
нажал на кнопку "Принять заказ", тогда у него в личном кабинете отобразится данный заказ, а также статус, который он
может изменять. После доставки заказ помечается как выполненный, а курьер может браться за новый.

### 3. Описание структуры базы данных

![БД](https://user-images.githubusercontent.com/98755619/213806083-25628793-9ee0-4597-a8bf-84e842209b1a.png)

### 4. Описание алгоритмов

### Алгоритм оформления доставки клиентом
![1 drawio (1)](https://user-images.githubusercontent.com/98755619/213801454-ac2981ff-77a0-4823-832b-7718dd357b26.png)

### Алгоритм регистрации на сайте как курьера
![2 drawio (3)](https://user-images.githubusercontent.com/98755619/213801551-8ba4c771-5b5a-493e-9c66-8fcf4d5690a8.png)

### Алгоритм принятия заказа курьером
![3 drawio](https://user-images.githubusercontent.com/98755619/213801603-418a9894-86eb-489d-8453-798e9da0ed43.png)

### 4. Значимые части кода

#### Модель Продуктов
```
class Product(models.Model):
    category = models.ForeignKey(
        Category,
        related_name='products',
        on_delete=models.CASCADE
    )
    name = models.CharField(max_length=200, db_index=True)
    slug = models.SlugField(max_length=200, db_index=True)
    image = models.ImageField(upload_to='products/', blank=True)
    description = models.TextField(blank=True)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    stock = models.PositiveIntegerField()
    available = models.BooleanField(default=True)
    created = models.DateTimeField(auto_now_add=True)
    updated = models.DateTimeField(auto_now=True)

    class Meta:
        verbose_name = 'Продукт'
        verbose_name_plural = 'Продукты'
        ordering = ('name',)
        index_together = (('id', 'slug'),)

    def __str__(self):
        return self.name

    def get_absolute_url(self):
        return reverse('orders:product_detail',
                       args=[self.id, self.slug])
```

#### Модель заказа

```
class Order(models.Model):
    WAITING_SUBMIT = 'WS'
    GOING_TO = 'GT'
    ON_THE_WAY = 'OTW'
    DELIVERED = 'DV'
    STATUS_CHOICES = (
        (WAITING_SUBMIT, 'Waiting submit'),
        (GOING_TO, 'Going to'),
        (ON_THE_WAY, 'On the way'),
        (DELIVERED, 'delivered'),
    )
    address = models.CharField(
        'Адрес получателя',
        max_length=400
    )
    client = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        verbose_name='Заказчик',
        related_name='all_orders'
    )
    pub_date = models.DateTimeField(
        'Время публикации',
        auto_now_add=True
    )
    status = models.CharField(
        'Статус заказа',
        max_length=40,
        choices=STATUS_CHOICES,
        default=WAITING_SUBMIT
    )
    comment = models.TextField(
        'Обращение к курьеру',
        max_length=2000,
        blank=True
    )
    paid = models.BooleanField(default=False)

    class Meta:
        verbose_name = 'Заказ'
        verbose_name_plural = 'Заказы'

    def __str__(self):
        return f'Order {self.id} of {self.client.username}'


```

#### Вью функция заказа

```
def new_order(request):
    cart = Cart(request)
    if request.method == 'POST':
        form = OrderForm(request.POST)
        if form.is_valid():
            order = form.save(commit=False)
            order.client = request.user
            order.paid = True
            order.save()
            for item in cart:
                OrderItem.objects.create(
                    order=order,
                    product=item['product'],
                    price=item['price'],
                    quantity=item['quantity']

                )
            cart.clear()
            return render(request, 'orders/sucess_order.html')
    form = OrderForm()
    return render(request, "orders/new_order.html", {'form': form})
```
