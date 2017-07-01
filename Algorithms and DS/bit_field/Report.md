# Методы программирования 2: Множества на основе битовых полей


## Цели и задачи

__Цель данной работы__  — разработка структуры данных для хранения множеств с
использованием битовых полей, а также освоение таких инструментов разработки
программного обеспечения, как система контроля версий [Git][git] и фрэймворк для
разработки автоматических тестов [Google Test][gtest].

Работа выполнялась на основе проекта-шаблона, содержащего следующее:

 - Интерфейсы классов битового поля и множества (h-файлы)
 - Готовый набор тестов для каждого из указанных классов
 - Пример использования класса битового поля и множества для решения задачи
   поиска простых чисел с помощью алгоритма ["Решето Эратосфена"][sieve]

Результатом выполнения работы является решение следующих задач:

  1. Реализация класса битового поля `TBitField` согласно заданному интерфейсу.
  1. Реализация класса множества `TSet` согласно заданному интерфейсу.
  1. Обеспечение работоспособности тестов и примера использования.
  1. Публикация исходных кодов в личном репозитории на BitBucket.

 __Реализация класса `TBitField`:__

```c++
#include "tbitfield.h"

TBitField::TBitField(int len)
{
	if (len > 0)
	{
		BitLen = len;
		MemLen = BitLen / TELbSize;
		if (BitLen % TELbSize > 0)
			MemLen++;
		pMem = new TELEM[MemLen];
		for (int i = 0; i < MemLen; i++)
			pMem[i] = 0;
	}
	else
		throw("incorrect data");
}

TBitField::TBitField(const TBitField &bf) // конструктор копирования
{
	BitLen = bf.BitLen;
	MemLen = bf.MemLen;
	pMem = new TELEM[MemLen];
	for (int i = 0; i < MemLen; i++)
		pMem[i] = bf.pMem[i];

}

TBitField::~TBitField()
{
	delete [] pMem;
	pMem = NULL;
}

int TBitField::GetMemIndex(const int n) const // индекс Мем для бита n
{
	if ((n >= 0) && (n < BitLen))
		return (n / TELbSize);
	else
		throw("incorrect data");
}

TELEM TBitField::GetMemMask(const int n) const // битовая маска для бита n
{
	if ((n >= 0) && (n < BitLen))
		return (1 << (n & (TELbSize - 1)));
	else
		throw("incorrect data");
}

// доступ к битам битового поля

int TBitField::GetLength(void) const // получить длину (к-во битов)
{
  return BitLen;
}

void TBitField::SetBit(const int n) // установить бит
{
	if ((n >= 0) && (n < BitLen))
	{
		pMem[GetMemIndex(n)] |= GetMemMask(n);
	}
	else
		throw("incorrect data");
}

void TBitField::ClrBit(const int n) // очистить бит
{
	if ((n >= 0) && (n < BitLen))
	{
		pMem[GetMemIndex(n)] &= ~GetMemMask(n);
	}
	else
		throw("incorrect data");
}

int TBitField::GetBit(const int n) const // получить значение бита
{
	if ((n >= 0) && (n < BitLen))
	{
		return pMem[GetMemIndex(n)] & GetMemMask(n);
	}
	else
		throw("incorrect data");
}

// битовые операции

TBitField& TBitField::operator=(const TBitField &bf) // присваивание
{
	if (pMem == NULL) 
		throw "incorrect data";
	if (this != &bf)
	{
		delete[] pMem;
		BitLen = bf.BitLen;
		MemLen = bf.MemLen;
		pMem = new TELEM[MemLen];
		for (int i = 0; i < MemLen; i++)
			pMem[i] = bf.pMem[i];
	}
	return *this;
}

int TBitField::operator==(const TBitField &bf) const // сравнение
{
	if (BitLen == bf.BitLen)
	{
		for (int i = 0; i < MemLen; i++)
			if (pMem[i] != bf.pMem[i])
				return 0;
		return 1;
	}
	return 0;
}

int TBitField::operator!=(const TBitField &bf) const // сравнение
{
  return (*this == bf)^1;
}

TBitField TBitField::operator|(const TBitField &bf) // операция "или"
{
	int len = BitLen;
	if (bf.BitLen > len)
		len = bf.BitLen;
	TBitField temp(len);
	for (int i = 0; i < MemLen; i++)
		temp.pMem[i] = pMem[i];
	for (int i = 0; i < bf.MemLen; i++)
		temp.pMem[i] |= bf.pMem[i];
	return temp;
}

TBitField TBitField::operator&(const TBitField &bf) // операция "и"
{
	int len = BitLen;
	if (bf.BitLen > len)
		len = bf.BitLen;
	TBitField temp(len);
	for (int i = 0; i < MemLen; i++)
		temp.pMem[i] = pMem[i];
	for (int i = 0; i < bf.MemLen; i++)
		temp.pMem[i] &= bf.pMem[i];
	return temp;
}

TBitField TBitField::operator~(void) // отрицание
{
	TBitField temp(*this);
	for (int i = 0; i < BitLen; i++)
	{
		if (!GetBit(i)) 
			temp.SetBit(i);
		else 
			temp.ClrBit(i);
	}
	return temp;
}

// ввод/вывод

istream &operator>>(istream &istr, TBitField &bf) // ввод
{
	char sign;
	int i = 1;
	istr >> sign;
	while (((sign == '0') || (sign == '1')) && (i < bf.BitLen - 1))
	{
		if (sign == '0')
			bf.ClrBit(i);
		else
			bf.SetBit(i);
		istr >> sign;
		i++;
	}
	return istr;
}

ostream &operator<<(ostream &ostr, const TBitField &bf) // вывод
{
	for (int i = bf.BitLen - 1; i >= 0; i--)
		if (bf.GetBit(i))
			ostr << 1;
		else
			ostr << 0;
	ostr << endl;
	return ostr;
}
```

__Реализация класса `TSet`:__  

```c++
#include "tset.h"

TSet::TSet(int mp) : BitField(mp)
{
	MaxPower = mp;
}

// конструктор копирования
TSet::TSet(const TSet &s) : BitField(s.BitField)
{
	MaxPower = s.MaxPower;
}

// конструктор преобразования типа
TSet::TSet(const TBitField &bf) : BitField(bf)
{
	MaxPower = bf.GetLength();
}

TSet::operator TBitField()
{
	return TBitField(MaxPower);
}

int TSet::GetMaxPower(void) const // получить макс. к-во эл-тов
{
	return MaxPower;
}

int TSet::IsMember(const int Elem) const // элемент множества?
{
		return BitField.GetBit(Elem);
}

void TSet::InsElem(const int Elem) // включение элемента множества
{
		BitField.SetBit(Elem);
}

void TSet::DelElem(const int Elem) // исключение элемента множества
{
		BitField.ClrBit(Elem);
}

// теоретико-множественные операции

TSet& TSet::operator=(const TSet &s) // присваивание
{
	if (this != &s)
	{
		MaxPower = s.MaxPower;
		BitField = s.BitField;
	}
	return *this;
}

int TSet::operator==(const TSet &s) const // сравнение
{
    return (MaxPower == s.MaxPower)&&(BitField == s.BitField);
}

int TSet::operator!=(const TSet &s) const // сравнение
{
	return ((*this == s) ^ 1);
}

TSet TSet::operator+(const TSet &s) // объединение
{
	TSet temp = (BitField | s.BitField);
	return temp;
}

TSet TSet::operator+(const int Elem) // объединение с элементом
{
		TSet temp(*this);
		temp.BitField.SetBit(Elem);
		return temp;
}

TSet TSet::operator-(const int Elem) // разность с элементом
{
		TSet temp(*this);
		temp.BitField.ClrBit(Elem);
		return temp;
}

TSet TSet::operator*(const TSet &s) // пересечение
{
	int U = (MaxPower > s.MaxPower) ? MaxPower : s.MaxPower;
	TSet temp(U);
	temp.BitField = BitField & s.BitField;
	return temp;
}

TSet TSet::operator~(void) // дополнение
{
	TSet temp(*this);
	temp.BitField = ~BitField;
	return temp;
}

// перегрузка ввода/вывода

istream &operator>>(istream &istr, TSet &s) // ввод
{
	char sign;
	int Elem = 0, pow = 0;
	istr >> sign;
	while ((sign != '\n') && (pow <= s.MaxPower))
	{
		while (sign != ' ')
		{
			if ((sign >= '0') && (sign <= '9'))
				Elem = Elem * 10 + (sign - '0');
			else
				throw("Incorrect data");
			istr >> sign;
		}
		s.InsElem(Elem);
		Elem = 0;
		pow++;
		istr >> sign;
	}
	return istr;
}

ostream& operator<<(ostream &ostr, const TSet &s) // вывод
{
	int i;
	ostr << "{ ";
	for (i = 0; i < s.MaxPower; i++)
		if (s.IsMember(i))
			ostr << i << ' ';
	ostr << '}' << endl;
	return ostr;
}
```

__Подтверждение успешного прохождения всех тестов:__

![gTest.jpg](http://i.imgur.com/qXnZNtD.jpg "gTest.jpg")

__Результат работы программы, основанной на алгоритме ["Решето Эратосфена"][sieve], использующей класс `TBitField` и `TSet` соответственно:__

![TBitField.jpg](http://i.imgur.com/1ecGC2M.jpg "TBitField.jpg")

![TSet.jpg](http://i.imgur.com/lVsX39l.jpg "TSet.jpg")
  
## Используемые инструменты

  - Система контроля версий [Git][git].
  - Фреймворк для написания автоматических тестов [Google Test][gtest].
  - Среда разработки Microsoft Visual Studio 2013.

##Вывод  
Основные цели работы достигнуты.  Классы `TBitField` и `TSet` успешно проходят предложенные тесты,  и их использование приводит к верным результатам при решении задачи по отысканию простых чисел. Ясно, что с помощью этих классов  можно решать и другие прикладные задачи.

Выполнение данной работы помогло мне освежить в памяти основные принципы создания классов в C++, дало основные навыки для взаимодействия с фреймворком  для написания автоматических тестов [Google Test][gtest].  На данный момент не ощущается острой необходимости в использовании системы контроля версий. Тем не менее, я старался фиксировать основные моменты. Ведь очевидно, что разработка серьезных программных систем невозможна без  данного инструмента, и умение с ним работать – это хороший задел на будущее.

<!-- LINKS -->

[git]:         https://git-scm.com/book/ru/v2
[gtest]:       https://github.com/google/googletest
[sieve]:       http://habrahabr.ru/post/91112
[git-guide]:   https://bitbucket.org/ashtan/mp2-lab1-set/src/ff6d76c3dcc2a531cefdc17aad5484c9bb8b47c5/docs/part1-git.md?at=master&fileviewer=file-view-default
[gtest-guide]: https://bitbucket.org/ashtan/mp2-lab1-set/src/ff6d76c3dcc2a531cefdc17aad5484c9bb8b47c5/docs/part2-google-test.md?at=master&fileviewer=file-view-default
[upstream]:    https://bitbucket.org/ashtan/mp2-lab1-set
