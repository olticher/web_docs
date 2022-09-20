#### Тесты
#### Тесты на модели

* Проверка атрибута destination
```
class TourModelCreateTest(TestCase):

    @classmethod
    def setUpTestData(cls):
        Tour.objects.create(
            id=1,
            destination="Направление",
            date_from=datetime.date.today(),
            date_to=datetime.date.today(),
            hotel="Название отеля",
            prev_price=1000,
            price=2000,
            count=1,
        )

    def test_tour_destination(self):
        tour = Tour.objects.get(id=1)
        self.assertEquals(tour.destination, "Направление")

```
* Проверка атрибута date_from

```
class TourModelFieldTypeTest(TestCase):

    @classmethod
    def setUpTestData(cls):
        Tour.objects.create(
            id=1,
            destination="Направление",
            date_from=datetime.date.today(),
            date_to=datetime.date.today(),
            hotel="Название отеля",
            prev_price=1000,
            price=2000,
            count=1,
        )

    def test_date_from_field_type(self):
        tour = Tour.objects.get(id=1)
        date_from = tour._meta.get_field('date_from')
        self.assertTrue(isinstance(date_from, DateField))
```

* Проверка строкового отображения

```
class TourModelStrTest(TestCase):

    @classmethod
    def setUpTestData(cls):
        Tour.objects.create(
            id=1,
            destination="Направление",
            date_from=datetime.date.today(),
            date_to=datetime.date.today(),
            hotel="Название отеля",
            prev_price=1000,
            price=2000,
            count=1,
        )

        CustomUser.objects.create(
            id=1,
            first_name="Имя",
            last_name="Фамилия",
        )

        Reservation.objects.create(
            id=1,
            user=CustomUser.objects.get(id=1),
            tour=Tour.objects.get(id=1),
            count=1,
            approved=True
        )

    def test_tour_empty_count(self):
        tour = Tour.objects.get(id=1)
        expected_value = tour.empty_count

        self.assertEquals(expected_value, 0)

    def test_tour_adjusted_empty_count(self):
        tour = Tour.objects.get(id=1)
        expected_value = tour.adjusted_empty_count(1)

        self.assertEquals(expected_value, 1)

    def test_tour_str(self):
        tour = Tour.objects.get(id=1)

        expected_tour_str = f"Тур \"{tour.destination}\" ({tour.date_from} - {tour.date_to})"

        self.assertEquals(str(tour), expected_tour_str)
```
Результат

![](/images/test_models.png)

#### Тесты на GET запросы

* Проверка получения тура

```
class GetTourTest(TestCase):

    @classmethod
    def setUpTestData(cls):
        Tour.objects.create(
            id=1,
            destination="Направление",
            date_from='2022-08-14',
            date_to='2022-08-21',
            hotel="Название отеля",
            prev_price=1000,
            price=2000,
            count=1,
        )

    def test_get_tour(self):
        url = reverse('tours:tours', args=['1'])

        data = {
            "count": 1,
            "next": None,
            "previous": None,
            'results': [{
                'date_from': '2022-08-14',
                'date_to': '2022-08-21',
                'destination': 'Направление',
                'empty_count': 1,
                'hotel': 'Название отеля',
                'id': 1,
                'prev_price': 1000,
                'price': 2000
            }]}

        response = self.client.get(url, format='json')
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(response.json(), data)

```

* Проверка получения резерваций

```
class SearchReservationsTest(TestCase):

    @classmethod
    def setUpTestData(cls):
        Tour.objects.create(
            id=1,
            destination="Направление",
            date_from='2022-08-14',
            date_to='2022-08-21',
            hotel="Название отеля",
            prev_price=1000,
            price=2000,
            count=1,
        )

        Tour.objects.create(
            id=2,
            destination="Направление2",
            date_from='2022-08-14',
            date_to='2022-08-21',
            hotel="Название отеля",
            prev_price=1000,
            price=2000,
            count=1,
        )

        CustomUser.objects.create(
            id=1,
            first_name="Имя",
            last_name="Фамилия",
        )

        Reservation.objects.create(
            id=1,
            user=CustomUser.objects.get(id=1),
            tour=Tour.objects.get(id=1),
            count=1,
            approved=True
        )

        Reservation.objects.create(
            id=2,
            user=CustomUser.objects.get(id=1),
            tour=Tour.objects.get(id=1),
            count=1,
            approved=True
        )

        Reservation.objects.create(
            id=3,
            user=CustomUser.objects.get(id=1),
            tour=Tour.objects.get(id=2),
            count=1,
            approved=True
        )

    def test_search_reservations(self):
        url = reverse('tours:reservations_search2')

        data = {
            "count": 1,
            "next": None,
            "previous": None,
            'results': [
                {
                    'approved': True,
                    'count': 1,
                    'has_review': False,
                    'id': 3,
                    'total_price': 2000,
                    'tour': {
                        'date_from': '2022-08-14',
                        'date_to': '2022-08-21',
                        'destination': 'Направление2',
                        'empty_count': 0,
                        'hotel': 'Название отеля',
                        'id': 2,
                        'prev_price': 1000,
                        'price': 2000}}
            ]
        }

        response = self.client.get(url, {'search': 'Направление2'},
                                   format='json')
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(response.json(), data)
```

* Проверка ранжирования стоимости тура

```
class GetPriceRangeFilteredToursTest(TestCase):

    @classmethod
    def setUpTestData(cls):
        Tour.objects.create(
            id=1,
            destination="Направление1",
            date_from='2022-08-14',
            date_to='2022-08-21',
            hotel="Название отеля 1",
            prev_price=1000,
            price=1000,
            count=1,
        )

        Tour.objects.create(
            id=2,
            destination="Направление2",
            date_from='2022-08-14',
            date_to='2022-08-21',
            hotel="Название отеля 2",
            prev_price=1000,
            price=2000,
            count=1,
        )

        Tour.objects.create(
            id=3,
            destination="Направление3",
            date_from='2022-08-14',
            date_to='2022-08-21',
            hotel="Название отеля 3",
            prev_price=1000,
            price=3000,
            count=1,
        )

    def test_price_range_filter_tours(self):
        url = reverse('tours:tours_price_range')

        data = {
            'count': 1,
            'links': {'next': None, 'previous': None},
            'num_pages': 1,
            'page_number': 1,
            'results': [
                {
                    'date_from': '2022-08-14',
                    'date_to': '2022-08-21',
                    'destination': 'Направление2',
                    'empty_count': 1,
                    'hotel': 'Название отеля 2',
                    'id': 2,
                    'prev_price': 1000,
                    'price': 2000
                }
            ]
        }

        response = self.client.get(url,
                                   {'price_min': '1500',
                                    'price_max': '2500',
                                    },
                                   format='json')
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(response.json(), data)

```

Результат

![](/images/test_GET.png)

#### Тесты на POST запросы

* Проверка создания тура

```
class CreateTourTest(TestCase):

    def test_create_tour(self):
        url = reverse('tours:create_tour')

        data = {
            'date_from': '2022-08-14',
            'date_to': '2022-08-21',
            'destination': 'Направление',
            'hotel': 'Название отеля',
            'id': 1,
            'prev_price': 1000,
            'price': 2000,
            'count': 0
        }

        response = self.client.post(url, data, format='json')
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        self.assertEqual(response.json(), data)
```

* Проверка резерваций тура

```
class CreateReservationTest(TestCase):

    @classmethod
    def setUpTestData(cls):
        Tour.objects.create(
            id=1,
            destination="Направление1",
            date_from='2022-08-14',
            date_to='2022-08-21',
            hotel="Название отеля 1",
            prev_price=1000,
            price=1000,
            count=1,
        )

        CustomUser.objects.create(
            id=1,
            first_name="Имя",
            last_name="Фамилия",
        )

    def test_create_reservation(self):
        url = reverse('tours:create_reservation')

        data = {
            'approved': True,
            'count': 1,
            'id': 1,
            'tour': 1,
            'user': 1
        }

        response = self.client.post(url, data, format='json')
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        self.assertEqual(response.json(), data)

```

* Проверка на создание отзыва к туру

```
class CreateReviewTest(TestCase):

    @classmethod
    def setUpTestData(cls):
        Tour.objects.create(
            id=1,
            destination="Направление1",
            date_from='2022-08-14',
            date_to='2022-08-21',
            hotel="Название отеля 1",
            prev_price=1000,
            price=1000,
            count=1,
        )

        CustomUser.objects.create(
            id=1,
            first_name="Имя",
            last_name="Фамилия",
        )

        Reservation.objects.create(
            id=1,
            user=CustomUser.objects.get(id=1),
            tour=Tour.objects.get(id=1),
            count=1,
            approved=True
        )

    def test_create_review(self):
        url = reverse('tours:create_review')

        data = {
            'id': 1,
            'reservation': 1,
            'text': 'Нравится!',
            'stars': 5,
        }

        response = self.client.post(url, data, format='json')
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        self.assertEqual(response.json(), data)
```

Результат

![](/images/test_POST.png)

#### Тесты на PATCH запросы

* Проверка на изменение отзыва к туру

```
class UpdateReviewTest(TestCase):

    @classmethod
    def setUpTestData(cls):
        Tour.objects.create(
            id=1,
            destination="Направление",
            date_from='2022-08-14',
            date_to='2022-08-21',
            hotel="Название отеля",
            prev_price=1000,
            price=2000,
            count=1,
        )

        CustomUser.objects.create(
            id=1,
            first_name="Имя",
            last_name="Фамилия",
        )

        Reservation.objects.create(
            id=1,
            user=CustomUser.objects.get(id=1),
            tour=Tour.objects.get(id=1),
            count=1,
            approved=True
        )

        Review.objects.create(
            id=1,
            reservation=Reservation.objects.get(id=1),
            text='Отзыв',
            stars=10
        )

    def test_update_review(self):
        get_url = reverse('tours:review', args=['1'])
        patch_url = reverse('tours:review', args=['1'])

        data = {
            'id': 1,
            'reservation': 1,
            'text': 'Отзыв',
            'stars': 10,
        }

        current_data = self.client.get(get_url, format='json')
        self.assertEqual(current_data.status_code, status.HTTP_200_OK)
        self.assertEqual(current_data.json(), data)

        data['stars'] = 8

        self.client.patch(patch_url, data, content_type='application/json')
        changed_data = self.client.get(get_url, format='json')
        self.assertEqual(changed_data.status_code, status.HTTP_200_OK)
        self.assertEqual(changed_data.json(), data)

```
* Проверка на изменение цены тура

```
class UpdateTourPriceTest(TestCase):

    @classmethod
    def setUpTestData(cls):
        Tour.objects.create(
            id=1,
            destination="Направление",
            date_from='2022-08-14',
            date_to='2022-08-21',
            hotel="Название отеля",
            prev_price=1000,
            price=2000,
            count=1,
        )

    def test_update_price(self):
        get_url = reverse('tours:tour', args=['1'])
        patch_url = reverse('tours:tour', args=['1'])

        data = {
            'date_from': '2022-08-14',
            'date_to': '2022-08-21',
            'destination': 'Направление',
            'hotel': 'Название отеля',
            'id': 1,
            'prev_price': 1000,
            'price': 2000,
            'count': 1
        }

        current_data = self.client.get(get_url, format='json')
        self.assertEqual(current_data.status_code, status.HTTP_200_OK)
        self.assertEqual(current_data.json(), data)

        data['price'] = 3000

        self.client.patch(patch_url, data,
                          content_type='application/json')

        data['prev_price'] = 2000

        changed_data = self.client.get(get_url, format='json')
        self.assertEqual(changed_data.status_code, status.HTTP_200_OK)
        self.assertEqual(changed_data.json(), data)
```
* Проверка на изменение подтверждения резервации

```
class UpdateReservationApprovalTest(TestCase):

    @classmethod
    def setUpTestData(cls):
        Tour.objects.create(
            id=1,
            destination="Направление",
            date_from='2022-08-14',
            date_to='2022-08-21',
            hotel="Название отеля",
            prev_price=1000,
            price=2000,
            count=1,
        )

        CustomUser.objects.create(
            id=1,
            first_name="Имя",
            last_name="Фамилия",
        )

        Reservation.objects.create(
            id=1,
            user=CustomUser.objects.get(id=1),
            tour=Tour.objects.get(id=1),
            count=1,
            approved=False
        )

    def test_update_approval(self):
        get_url = reverse('tours:reservation2', args=['1'])
        patch_url = reverse('tours:reservation2', args=['1'])

        data = {
            'approved': False,
            'count': 1,
            'id': 1,
            'tour': 1,
            'user': 1
        }

        current_data = self.client.get(get_url, format='json')
        self.assertEqual(current_data.status_code, status.HTTP_200_OK)
        self.assertEqual(current_data.json(), data)

        data['approved'] = True

        self.client.patch(patch_url, data, content_type='application/json')
        changed_data = self.client.get(get_url, format='json')
        self.assertEqual(changed_data.status_code, status.HTTP_200_OK)
        self.assertEqual(changed_data.json(), data)
```
Результат

![](/images/test_PATCH.png)