# Методы программирования 2: Верхнетреугольные матрицы на шаблонах


## Цели и задачи

В рамках лабораторной работы была поставлена задача создания программных средств, поддерживающих эффективное хранение матриц специального вида (верхнетреугольных) и выполнение основных операций над ними:

- сложение/вычитание;
- копирование;
- сравнение.

Работа выполнялась на основе проекта-шаблона, содержащего следующее:

 - Интерфейсы классов Вектор и Матрица (h-файл)
 - Начальный набор готовых тестов для каждого из указанных классов.
 - Набор заготовок тестов для каждого из указанных классов. 
 - Тестовый пример использования класса Матрица

Результатом выполнения работы является решение следующих задач:

  1. Реализация методов шаблонного класса `TVector` согласно заданному интерфейсу.
  1. Реализация методов шаблонного класса `TMatrix` согласно заданному интерфейсу.
  1. Обеспечение работоспособности тестов и примера использования.
  1. Реализация заготовок тестов, покрывающих все методы классов `TVector` и `TMatrix`.
  1. Модификация примера использования в тестовое приложение, позволяющее задавать матрицы и осуществлять основные операции над ними.

 __Реализация класса `TVector` и `TMatrix`:__

```c++
#ifndef __TMATRIX_H__
#define __TMATRIX_H__

#include <iostream>

using namespace std;

const int MAX_VECTOR_SIZE = 100000000;
const int MAX_MATRIX_SIZE = 10000;

// Шаблон вектора
template <class ValType>
class TVector
{
protected:
  ValType *pVector;
  int Size;       // размер вектора
  int StartIndex; // индекс первого элемента вектора
public:
  TVector(int s = 10, int si = 0);
  TVector(const TVector &v);                // конструктор копирования
  ~TVector();
  int GetSize()      { return Size;       } // размер вектора
  int GetStartIndex(){ return StartIndex; } // индекс первого элемента
  ValType& operator[](int pos);             // доступ
  bool operator==(const TVector &v) const;  // сравнение
  bool operator!=(const TVector &v) const;  // сравнение
  TVector& operator=(const TVector &v);     // присваивание

  // скалярные операции
  TVector  operator+(const ValType &val);   // прибавить скаляр
  TVector  operator-(const ValType &val);   // вычесть скаляр
  TVector  operator*(const ValType &val);   // умножить на скаляр

  // векторные операции
  TVector  operator+(const TVector &v);     // сложение
  TVector  operator-(const TVector &v);     // вычитание
  ValType  operator*(const TVector &v);     // скалярное произведение

  // ввод-вывод
  friend istream& operator>>(istream &in, TVector &v)
  {
    for (int i = 0; i < v.Size; i++)
      in >> v.pVector[i];
    return in;
  }
  friend ostream& operator<<(ostream &out, const TVector &v)
  {
    for (int i = 0; i < v.Size; i++)
      out << v.pVector[i] << ' ';
    return out;
  }
};

template <class ValType>
TVector<ValType>::TVector(int s, int si)
{
	if ((s >= 0)&&(si > 0)&&(s <= MAX_VECTOR_SIZE)&&(si <= MAX_VECTOR_SIZE))
		{
			Size = s;
			StartIndex = si;
			pVector = new ValType[Size];
		}
	else
		throw("incorrect data");
} /*-------------------------------------------------------------------------*/

template <class ValType> //конструктор копирования
TVector<ValType>::TVector(const TVector<ValType> &v)
{
	Size = v.Size;
	StartIndex = v.StartIndex;
	pVector = new ValType[Size];
	for (int i = 0; i < Size; i++)
		pVector[i] = v.pVector[i];
} /*-------------------------------------------------------------------------*/

template <class ValType>
TVector<ValType>::~TVector()
{
	delete [] pVector;
} /*-------------------------------------------------------------------------*/

template <class ValType> // доступ
ValType& TVector<ValType>::operator[](int pos)
{
	if ((pos >= StartIndex) && (pos <= Size + StartIndex - 1))
		return pVector[pos - StartIndex];
	else
		throw("Incorrect data");
} /*-------------------------------------------------------------------------*/

template <class ValType> // сравнение
bool TVector<ValType>::operator==(const TVector &v) const
{
	if (Size != v.Size) return false;
	for (int i = 0; i < Size; i++)
		if (pVector[i] != v.pVector[i])
			return false;
	return true;
} /*-------------------------------------------------------------------------*/

template <class ValType> // сравнение
bool TVector<ValType>::operator!=(const TVector &v) const
{
	return !(*this == v);
} /*-------------------------------------------------------------------------*/

template <class ValType> // присваивание
TVector<ValType>& TVector<ValType>::operator=(const TVector &v)
{
	if (this != &v)
	{
		delete [] pVector;
		Size = v.Size;
		StartIndex = v.StartIndex;
		pVector = new ValType[Size];
		for (int i = 0; i < Size; i++)
			pVector[i] = v.pVector[i];
		return *this;
	}
	else
		return *this;
} /*-------------------------------------------------------------------------*/

template <class ValType> // прибавить скаляр
TVector<ValType> TVector<ValType>::operator+(const ValType &val)
{
	TVector<ValType> temp(*this);
	for (int i = 0; i < Size; i++)
		temp.pVector[i] = temp.pVector[i] + val;
	return temp;
} /*-------------------------------------------------------------------------*/

template <class ValType> // вычесть скаляр
TVector<ValType> TVector<ValType>::operator-(const ValType &val)
{
	TVector<ValType> temp(*this);
	for (int i = 0; i < Size; i++)
		temp.pVector[i] = temp.pVector[i] - val;
	return temp;
} /*-------------------------------------------------------------------------*/

template <class ValType> // умножить на скаляр
TVector<ValType> TVector<ValType>::operator*(const ValType &val)
{
	TVector<ValType> temp(*this);
	for (int i = 0; i < Size; i++)
		temp.pVector[i] = temp.pVector[i] * val;
	return temp;
} /*-------------------------------------------------------------------------*/

template <class ValType> // сложение
TVector<ValType> TVector<ValType>::operator+(const TVector<ValType> &v)
{
	if (Size == v.Size)
	{
		TVector<ValType> temp(*this);
		for (int i = 0; i < Size; i++)
			temp.pVector[i] = pVector[i] + v.pVector[i];
		return temp;
	}
	else
		throw("Incorrect data");
} /*-------------------------------------------------------------------------*/

template <class ValType> // вычитание
TVector<ValType> TVector<ValType>::operator-(const TVector<ValType> &v)
{
	if (Size == v.Size)
	{
		TVector<ValType> temp(*this);
		for (int i = 0; i < Size; i++)
			temp.pVector[i] = pVector[i] - v.pVector[i];
		return temp;
	}
	else
		throw("Incorrect data");
} /*-------------------------------------------------------------------------*/

template <class ValType> // скалярное произведение
ValType TVector<ValType>::operator*(const TVector<ValType> &v)
{
	ValType rez = 0;
	if (Size == v.Size)
	{
	for (int i = 0; i < Size; i++)
		rez += pVector[i] * v.pVector[i];
	}
	else
		throw("Incorrect data");
	return rez;

} /*-------------------------------------------------------------------------*/


// Верхнетреугольная матрица
template <class ValType>
class TMatrix : public TVector<TVector<ValType> >
{
public:
  TMatrix(int s = 10);                           
  TMatrix(const TMatrix &mt);                    // копирование
  TMatrix(const TVector<TVector<ValType> > &mt); // преобразование типа
  bool operator==(const TMatrix &mt) const;      // сравнение
  bool operator!=(const TMatrix &mt) const;      // сравнение
  TMatrix& operator= (const TMatrix &mt);        // присваивание
  TMatrix  operator+ (const TMatrix &mt);        // сложение
  TMatrix  operator- (const TMatrix &mt);        // вычитание

  // ввод / вывод
  friend istream& operator>>(istream &in, TMatrix &mt)
  {
    for (int i = 0; i < mt.Size; i++)
      in >> mt.pVector[i];
    return in;
  }
  friend ostream & operator<<( ostream &out, const TMatrix &mt)
  {
    for (int i = 0; i < mt.Size; i++)
      out << mt.pVector[i] << endl;
    return out;
  }
};

template <class ValType>
TMatrix<ValType>::TMatrix(int s): TVector<TVector<ValType> >(s)
{
	if ((s >= 0) && (s <= MAX_MATRIX_SIZE))
		for (int i = 0; i < s; i++)
			pVector[i] = TVector<ValType>(s - i, i);

	else
		throw("Incorrect data");
} /*-------------------------------------------------------------------------*/

template <class ValType> // конструктор копирования
TMatrix<ValType>::TMatrix(const TMatrix<ValType> &mt):
  TVector<TVector<ValType> >(mt) {}

template <class ValType> // конструктор преобразования типа
TMatrix<ValType>::TMatrix(const TVector<TVector<ValType> > &mt):
  TVector<TVector<ValType> >(mt) {}

template <class ValType> // сравнение
bool TMatrix<ValType>::operator==(const TMatrix<ValType> &mt) const
{
	return TVector<TVector<ValType> >::operator==(mt);
} /*-------------------------------------------------------------------------*/

template <class ValType> // сравнение
bool TMatrix<ValType>::operator!=(const TMatrix<ValType> &mt) const
{
	return TVector<TVector<ValType> >::operator!=(mt);
} /*-------------------------------------------------------------------------*/

template <class ValType> // присваивание
TMatrix<ValType>& TMatrix<ValType>::operator=(const TMatrix<ValType> &mt)
{
	TVector<TVector<ValType> >::operator=(mt);
	return *this;
} /*-------------------------------------------------------------------------*/

template <class ValType> // сложение
TMatrix<ValType> TMatrix<ValType>::operator+(const TMatrix<ValType> &mt)
{
	return TVector<TVector<ValType> >::operator+(mt);
} /*-------------------------------------------------------------------------*/

template <class ValType> // вычитание
TMatrix<ValType> TMatrix<ValType>::operator-(const TMatrix<ValType> &mt)
{
	return TVector<TVector<ValType> >::operator-(mt);
} /*-------------------------------------------------------------------------*/

#endif
```

__Подтверждение успешного прохождения всех тестов:__

![gTest.jpg](http://i.imgur.com/edez4xi.jpg "gTest.jpg")

__Результат работы предложенной программы, которая использует класс `TMatrix`:__

![TBitField.jpg](http://i.imgur.com/V9fe1kv.jpg "TBitField.jpg")

##Собственный пример использования

Мною был реализован алгоритм, использующий верхнетреугольные матрицы и основывающийся на принципе треугольной последовательности биномиальных коэффициентов. Программа для введенного с клавиатуры n печатает последовательность биноминальных коэфициентов данной степени, выводит на экран треугольник Паскаля и проверяет свойство этой таблицы (сумма элементов n-ой строки = 2^n).

 __Реализация собственного примера использования:__  

```c++
// Тестирование верхнетреугольной матрицы

#include <iostream>
#include "utmatrix.h"
//---------------------------------------------------------------------------

int main()
{
	setlocale(LC_ALL, "Russian");
	unsigned int n;
	cout << "Тестирование класса TMatrix на примере треугольника Паскаля и биноминальных коэфициентов" << '\n' << endl;
	cout << "Введите степень n >>" << '\n' << endl;
	cin >> n;
	cout << '\n';
	TMatrix<int> triangle(n + 1);
	int i, j, sum = 0, t_Size = triangle.GetSize();
	triangle[t_Size - 1][t_Size - 1] = 1;
	for (i = t_Size - 2; i >= 0; i--)
	{
		for (j = i; j < t_Size; j++)
		if ((j == i) || (j == t_Size - 1))
			triangle[i][j] = 1;
		else
			triangle[i][j] = triangle[i + 1][j + 1] + triangle[i + 1][j];
	}
	for (i = 0; i < t_Size; i++)
		sum += triangle[0][i];
	cout << "Биноминальные коэфициенты для " << n << " степени: " << '\n' << triangle[0] << '\n' << endl;
	cout << "Треугольник Паскаля: \n" << triangle << '\n' << endl;
	cout << "Сумма коэфициентов строки с номером " << n << " : " << sum << " = 2^" << n << endl;
	return 0;
}
//---------------------------------------------------------------------------
```
Нумерация осуществляется снизу вверх. Результат работы программы для n = 15:

![Own_Sample.jpg](http://i.imgur.com/jSI1HfL.jpg "Own_Sample.jpg")

Возможно, использование класса 'TMatrix' - не самый лучший выбор для решения подобной задачи. С одной стороны, мы не используем лишнюю память и храним лишь нужные элементы. Но с другой, нагружаем систему частыми обращаниями к методам класса, что эквиваленто многократным вызовам функций. 

## Используемые инструменты

  - Система контроля версий [Git][git].
  - Фреймворк для написания автоматических тестов [Google Test][gtest].
  - Среда разработки Microsoft Visual Studio 2013.

##Вывод  
Основные цели работы достигнуты.  Классы `TVector` и `TMatrix` успешно проходят предложенные тесты, включая те, что были предложены написать мне, показывают корректную работу в программах-примерах.

Выполнение данной работы помогло мне освежить в памяти основные принципы наследования классов в C++, закрепило знания о фреймворке для написания автоматических тестов [Google Test][gtest].

<!-- LINKS -->

[git]:         https://git-scm.com/book/ru/v2
[gtest]:       https://github.com/google/googletest
[sieve]:       http://habrahabr.ru/post/91112
[git-guide]:   https://bitbucket.org/ashtan/mp2-lab1-set/src/ff6d76c3dcc2a531cefdc17aad5484c9bb8b47c5/docs/part1-git.md?at=master&fileviewer=file-view-default
[gtest-guide]: https://bitbucket.org/ashtan/mp2-lab1-set/src/ff6d76c3dcc2a531cefdc17aad5484c9bb8b47c5/docs/part2-google-test.md?at=master&fileviewer=file-view-default
[upstream]:    https://bitbucket.org/ashtan/mp2-lab1-set

