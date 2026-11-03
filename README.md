"""Домашнее задание «ООП: наследование, инкапсуляция и полиморфизм»."""


class Mentor:
    """Родительский класс: лекторы и проверяющие."""

    def __init__(self, name, surname):
        self.name = name
        self.surname = surname
        self.courses_attached = []


class Lecturer(Mentor):
    """Лектор: получает оценки за лекции от студентов."""

    def __init__(self, name, surname):
        super().__init__(name, surname)
        self.grades = {}

    def __str__(self):
        return (f'Имя: {self.name}\nФамилия: {self.surname}\n'
                f'Средняя оценка за лекции: {self.avg_grade():.1f}')

    def avg_grade(self):
        """Средняя оценка лектора по всем полученным оценкам."""
        all_grades = [grade for grades in self.grades.values() for grade in grades]
        return sum(all_grades) / len(all_grades) if all_grades else 0

    def __eq__(self, other):
        if not isinstance(other, Lecturer):
            return NotImplemented
        return self.avg_grade() == other.avg_grade()

    def __lt__(self, other):
        if not isinstance(other, Lecturer):
            return NotImplemented
        return self.avg_grade() < other.avg_grade()


class Reviewer(Mentor):
    """Проверяющий: выставляет студентам оценки за домашние задания."""

    def rate_hw(self, student, course, grade):
        """Ставит студенту оценку за ДЗ по закреплённому курсу."""
        if isinstance(student, Student) and course in self.courses_attached \
                and course in student.courses_in_progress:
            student.grades.setdefault(course, []).append(grade)
        else:
            return 'Ошибка'

    def __str__(self):
        return f'Имя: {self.name}\nФамилия: {self.surname}'


class Student:
    """Студент: учится, оценивает лекторов."""

    def __init__(self, name, surname, gender):
        self.name = name
        self.surname = surname
        self.gender = gender
        self.finished_courses = []
        self.courses_in_progress = []
        self.grades = {}

    def rate_lecture(self, lecturer, course, grade):
        """Ставит лектору оценку за лекцию (от 1 до 10)."""
        if not isinstance(lecturer, Lecturer):
            return 'Ошибка'
        if course not in self.courses_in_progress \
                or course not in lecturer.courses_attached:
            return 'Ошибка'
        if not 1 <= grade <= 10:
            return 'Ошибка'
        lecturer.grades.setdefault(course, []).append(grade)

    def avg_grade(self):
        """Средняя оценка студента по всем ДЗ."""
        all_grades = [grade for grades in self.grades.values() for grade in grades]
        return sum(all_grades) / len(all_grades) if all_grades else 0

    def __str__(self):
        courses_in_progress = ', '.join(self.courses_in_progress)
        finished_courses = ', '.join(self.finished_courses)
        return (f'Имя: {self.name}\nФамилия: {self.surname}\n'
                f'Средняя оценка за домашние задания: {self.avg_grade():.1f}\n'
                f'Курсы в процессе изучения: {courses_in_progress}\n'
                f'Завершенные курсы: {finished_courses}')

    def __eq__(self, other):
        if not isinstance(other, Student):
            return NotImplemented
        return self.avg_grade() == other.avg_grade()

    def __lt__(self, other):
        if not isinstance(other, Student):
            return NotImplemented
        return self.avg_grade() < other.avg_grade()


def avg_hw_grade(students, course):
    """Средняя оценка за ДЗ по всем студентам в рамках конкретного курса."""
    grades = []
    for student in students:
        grades += student.grades.get(course, [])
    return round(sum(grades) / len(grades), 1) if grades else 0


def avg_lecture_grade(lecturers, course):
    """Средняя оценка за лекции всех лекторов в рамках курса."""
    grades = []
    for lecturer in lecturers:
        grades += lecturer.grades.get(course, [])
    return round(sum(grades) / len(grades), 1) if grades else 0


# Проверка наследования (задание № 1)
lecturer = Lecturer('Иван', 'Иванов')
reviewer = Reviewer('Пётр', 'Петров')
print(isinstance(lecturer, Mentor))  # True
print(isinstance(reviewer, Mentor))  # True
print(lecturer.courses_attached)     # []
print(reviewer.courses_attached)     # []

# Проверка метода rate_lecture (задание № 2)
student = Student('Алёхина', 'Ольга', 'Ж')

student.courses_in_progress += ['Python', 'Java']
lecturer.courses_attached += ['Python', 'C++']
reviewer.courses_attached += ['Python', 'C++']

print(student.rate_lecture(lecturer, 'Python', 7))   # None
print(student.rate_lecture(lecturer, 'Java', 8))     # Ошибка
print(student.rate_lecture(lecturer, 'C++', 8))      # Ошибка
print(student.rate_lecture(reviewer, 'Python', 6))   # Ошибка

print(lecturer.grades)  # {'Python': [7]}

# Проверка rate_hw проверяющего (задание № 2)
reviewer.rate_hw(student, 'Python', 10)
print(student.grades)  # {'Python': [10]}

# Проверка __str__ (задание № 3)
print(reviewer)
print(lecturer)
print(student)

# Проверка сравнения (задание № 3)
second_lecturer = Lecturer('Анна', 'Смирнова')
second_lecturer.courses_attached += ['Python']
second_lecturer.grades = {'Python': [9, 10]}
print(lecturer == second_lecturer)   # False (средняя 7.0 против 9.5)
print(lecturer < second_lecturer)    # True

second_student = Student('Ruoy', 'Eman', 'М')
second_student.courses_in_progress += ['Python', 'Git']
second_student.finished_courses += ['Введение в программирование']
reviewer.rate_hw(second_student, 'Python', 9)
reviewer.rate_hw(second_student, 'Python', 10)
reviewer.rate_hw(second_student, 'Git', 8)
reviewer.rate_hw(second_student, 'Git', 10)
print(second_student < student)  # сравнение по средним оценкам

# Полевые испытания (задание № 4)
print(avg_hw_grade([student, second_student], 'Python'))      # 9.5
print(avg_lecture_grade([lecturer, second_lecturer], 'Python'))  # 8.0