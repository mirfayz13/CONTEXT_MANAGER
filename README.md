# CONTEXT_MANAGER
import psycopg2

class DbConnect:
    def __init__(self, db_params):
        self.db_params = db_params
        self.conn = psycopg2.connect(**self.db_params)

    def __enter__(self):
        self.cur = self.conn.cursor()
        return self.cur

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.conn.commit()
        self.conn.close()

class Person:
    def __init__(self,
                 id: int | None = None,
                 full_name: str | None = None,
                 age: str | None = None,
                 email: str | None = None ):
        self.id = id
        self.full_name =full_name
        self.age = age
        self.email = email


    def get_all(self):
        with DbConnect(db_params) as cur:
            select_query = 'SELECT * FROM persons;'
            cur.execute(select_query)
            persons = [Person(id=row[0], full_name=row[1], age=row[2], email=row[3]) for row in cur.fetchall()]
            return persons

    def save(self):
        with DbConnect(db_params) as cur:
            insert_query = 'INSERT INTO people (full_name, age, email) VALUES (%s, %s, %s);'
            insert_params = (self.full_name, self.age, self.email)
            cur.execute(insert_query, insert_params)


    def __repr__(self):
        return f'Person({self.id} => {self.full_name} => {self.age} => {self.email})'

    @staticmethod
    def get_person(person_id):
        with DbConnect(db_params) as cur:
            cur.execute("SELECT * FROM people WHERE id = %s", (person_id,))
            person_data = cur.fetchone()
            if person_data:
                return Person(id=person_data[0], full_name=person_data[1], age=person_data[2], email=person_data[3])
            else:
                return None

db_params = {
    "dbname": input("DBNAME: "),
    "user": input("USER: "),
    "password": input("PASSWORD: "),
    "host": input("HOST: "),
    "port": input("PORT: ")
}

DbConnect(db_params)

while True:
    print("1. ADD PERSON")
    print("2. VIEW ALL PEOPLE")
    print("3. FIND PERSON")
    print("4. EXIT ")
    choice = input("SELECT: ")

    if choice == '1':
        full_name = input("FULL NAME: ")
        age = input("AGE: ")
        email = input("EMAIL: ")
        person = Person(full_name=full_name, age=age, email=email)
        person.save()
        print("PERSON IS SUCCESSFULLY ADDED")
    elif choice == '2':
        persons = Person().get_all()
        if persons:
            for person in persons:
                print(person)
        else:
            print("PERSON DOES NOT EXIST")
    elif choice == '3':
        person_id = int(input("USER ID : "))
        person = Person.get_person(person_id)
        if person:
            print(person)
        else:
            print("COULD NOT FIND USER WITH THIS ID")
    elif choice == '4':
        print("EXIT ")
        break
    else:
        print("ERROR")
