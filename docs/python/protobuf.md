https://github.com/google/protobuf/releases

```
$ ./configure
$ sudo make
$ sudo make check
$ sudo make install
```

编译时间超长

protoc: error while loading shared libraries: libprotoc.so.16: cannot open shared object file: No such file or directory


https://stackoverflow.com/questions/25518701/protobuf-cannot-find-shared-libraries

```
pipenv install protobuf
```

```
protoc -I=. --python_out=. ./addressbook.proto
```

``` python
import addressbook_pb2

address_book = addressbook_pb2.AddressBook()

person = address_book.person.add()
person.id = 1234
person.name = "John Doe"
person.email = "jdoe@example.com"

phone = person.phone.add()
phone.number = "555-4321"
phone.type = addressbook_pb2.Person.HOME

phone2 = person.phone.add()
phone2.number = "12324234"
phone2.type = addressbook_pb2.Person.WORK




person = address_book.person.add()
person.id = 1234
person.name = "John Doe"
person.email = "jdoe@example.com"

phone = person.phone.add()
phone.number = "555-4321"
phone.type = addressbook_pb2.Person.HOME

phone2 = person.phone.add()
phone2.number = "12324234"
phone2.type = addressbook_pb2.Person.WORK



data = address_book.SerializeToString()

print(data)

address_book.ParseFromString(data)

print(address_book)
```