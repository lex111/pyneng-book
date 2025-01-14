Протокол итерации
~~~~~~~~~~~~~~~~~


**Итерируемый объект (iterable)** - это объект, который способен возвращать элементы по одному.
Кроме того, это объект, из которого можно получить итератор.

В Python за получение итератора отвечает функция iter():

.. code:: python

    In [1]: lista = [1, 2, 3]

    In [2]: iter(lista)
    Out[2]: <list_iterator at 0xb4ede28c>

Функция ``iter`` отработает на любом объекте, у которого есть метод __iter__ или метод __getitem__.
Метод __iter__ возвращает итератор. Если этого метода нет, функция iter() проверяет, нет ли метода __getitem__ - метода, который позволяет получать элементы по индексу.
Если метод __getitem__ есть, возвращается итератор, который проходится по элементам, используя индекс (начиная с 0).


**Итератор (iterator)** - это объект, который возвращает свои элементы по одному за раз.
С точки зрения Python - это любой объект, у которого есть метод __next__. Этот метод возвращает следующий элемент, если он есть, или возвращает исключение StopIteration, когда элементы закончились.
Кроме того, итератор запоминает, на каком объекте он остановился в последнюю итерацию.
Также у каждого итератора присутствует метод __iter__ - то есть, любой итератор является итерируемым объектом. Этот метод возвращает сам итератор.

Пример создания итератора из списка:

.. code:: python

    In [3]: lista = [1, 2, 3]

    In [4]: i = iter(lista)

Теперь можно использовать функцию next(), которая вызывает метод __next__, чтобы взять следующий элемент:

.. code:: python

    In [5]: next(i)
    Out[5]: 1

    In [6]: next(i)
    Out[6]: 2

    In [7]: next(i)
    Out[7]: 3

    In [8]: next(i)
    ------------------------------------------------------------
    StopIteration              Traceback (most recent call last)
    <ipython-input-8-bed2471d02c1> in <module>()
    ----> 1 next(i)

    StopIteration:

После того, как элементы закончились, возвращается исключение StopIteration.
Для того, чтобы итератор снова начал возвращать элементы, его надо заново создать.
Аналогичные действия выполяются, когда цикл for проходится по списку:

.. code:: python

    In [9]: for item in lista:
       ...:     print(item)
       ...:
    1
    2
    3

Когда мы перебираем элементы списка, к списку сначала применяется функция iter(), чтобы создать итератор, а затем вызывается его метод __next__ до тех пор, пока не возникнет исключение StopIteration.

Создание итератора
^^^^^^^^^^^^^^^^^^

Пример класса Network:

.. code:: python

    In [10]: import ipaddress
        ...:
        ...: class Network:
        ...:     def __init__(self, network):
        ...:         self.network = network
        ...:         subnet = ipaddress.ip_network(self.network)
        ...:         self.addresses = [str(ip) for ip in subnet.hosts()]

Пример создания экземпляра класса Network:

.. code:: python

    In [14]: net1 = Network('10.1.1.192/30')

    In [15]: net1
    Out[15]: <__main__.Network at 0xb3124a6c>

    In [16]: net1.addresses
    Out[16]: ['10.1.1.193', '10.1.1.194']

    In [17]: net1.network
    Out[17]: '10.1.1.192/30'

Создаем итератор из класса Network:

.. code:: python

    In [12]: class Network:
        ...:     def __init__(self, network):
        ...:         self.network = network
        ...:         subnet = ipaddress.ip_network(self.network)
        ...:         self.addresses = [str(ip) for ip in subnet.hosts()]
        ...:         self._index = 0
        ...:
        ...:     def __iter__(self):
        ...:         print('Вызываю __iter__')
        ...:         return self
        ...:
        ...:     def __next__(self):
        ...:         print('Вызываю __next__')
        ...:         if self._index < len(self.addresses):
        ...:             current_address = self.addresses[self._index]
        ...:             self._index += 1
        ...:             return current_address
        ...:         else:
        ...:             raise StopIteration
        ...:

Метод __iter__ в итераторе должен возвращать сам объект, поэтому в методе 
указано ``return self``, а метод __next__ возвращает элементы по одному и генерирует 
исключение StopIteration, когда элементы закончились.


.. code:: python

    In [14]: net1 = Network('10.1.1.192/30')

    In [15]: for ip in net1:
        ...:     print(ip)
        ...:
    Вызываю __iter__
    Вызываю __next__
    10.1.1.193
    Вызываю __next__
    10.1.1.194
    Вызываю __next__

Чаще всего, итератор это одноразовый объект и перебрав элементы, мы уже не можем это сделать второй раз:

.. code:: python

    In [16]: for ip in net1:
        ...:     print(ip)
        ...:
    Вызываю __iter__
    Вызываю __next__


Создание итерируемого объекта
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Очень часто классу достаточно быть итерируемым объектом и не обязательно быть итератором.
Если объект будет итерируемым, его можно использовать в цикле for,
функциях map, filter, sorted, enumerate и других. Также, как правило, объект проще сделать итерируемым, чем итератором.

Для того чтобы класс Network создавал итерируемые объекты, надо чтобы в классе был метод
__iter__ (__next__ не нужен) и чтобы метод возвращал итератор.
Так как в данном случае, Network перебирает адреса, которые находятся в списке self.addresses,
самый просто вариант возвращать итератор, это вернуть ``iter(self.addresses)``:

.. code:: python

    In [17]: class Network:
        ...:     def __init__(self, network):
        ...:         self.network = network
        ...:         subnet = ipaddress.ip_network(self.network)
        ...:         self.addresses = [str(ip) for ip in subnet.hosts()]
        ...:
        ...:     def __iter__(self):
        ...:         return iter(self.addresses)
        ...:

Теперь все экземпляры класса Network будут итерируемыми объектами:

.. code:: python

    In [18]: net1 = Network('10.1.1.192/30')

    In [19]: for ip in net1:
        ...:     print(ip)
        ...:
    10.1.1.193
    10.1.1.194

