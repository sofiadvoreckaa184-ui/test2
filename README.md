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