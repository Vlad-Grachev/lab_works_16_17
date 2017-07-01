# Лабораторная работа №5. Полиномы #


## Цели и задачи

В рамках лабораторной работы ставится задача создания программных средств, поддерживающих эффективное представление полиномов и выполнение следующих операций над ними:

*	ввод полинома 
*	организация хранения полинома
*	удаление введенного ранее полинома;
*	копирование полинома;
*	сложение двух полиномов;
* 	вычисление значения полинома при заданных значениях переменных;
*	вывод.

В качестве структуры хранения используются циклические списки с заголовком. В числе операций над списками реализованы следующие действия:

*	поддержка понятия текущего звена;
*	вставка звеньев в начало, после текущей позиции и в конец списков;
*	удаление звеньев в начале и в текущей позиции списков;
*	организация последовательного доступа к звеньям списка (итератор).

### Условия и ограничения

При выполнении лабораторной работы использовались следующие основные предположения:

1.	Разработка структуры хранения ориентирована на представление полиномов от трех неизвестных.
2.	Степени переменных полиномов не могут превышать значения 9, т.е. 0 <= i, j, k <= 9.
3.	Число мономов в полиномах существенно меньше максимально возможного количества (тем самым, в структуре хранения должны находиться только мономы с ненулевыми коэффициентами).


**Результатом выполнения работы является следующий набор файлов:**

-	DatValue.h – модуль, объявляющий абстрактный класс объектов-значений списка;
-	Monom.h – модуль класса моном;
-	RootLink.h – модуль базового класса для звеньев (элементов) списка;
-	DatLink.h, DatLink.cpp – модуль класса для звеньев (элементов) списка с указателем на объект-значение;
-	DatList.h, DatList.cpp – модуль класса линейных списков;
-	HeadRing.h, HeadRing.cpp – модуль класса циклических списков с заголовком;
-	Polinom.h, Polinom.cpp – модуль класса полиномов;
-   poly_test_kit.cpp – модуль программы тестирования.

 
 __Реализация класса `TDatValue`:__

```c++
//DatValue.h
// модуль, объявляющий абстрактный класс объектов-значений списка

#ifndef _DATVALUE_H_
#define _DATVALUE_H_

class TDatValue;
typedef TDatValue *PTDatValue;

class TDatValue {
public:
	virtual TDatValue * GetCopy() = 0; // создание копии
	~TDatValue() {}
};
#endif
```
 __Реализация класса `TMonom`:__

```c++
// Monom.h
// модуль класса моном

#ifndef _MONOM_H_
#define _MONOM_H_

#include "DatValue/DatValue.h"

class TMonom;
typedef TMonom *PTMonom;

class TMonom : public TDatValue  {
protected:
	int Coeff; // коэффициент монома
	int Index; // индекс (свертка степеней)
public:
	TMonom(int cval = 1, int ival = 0) {
		Coeff = cval; Index = ival;
	};
	void SetCoeff(int cval) { Coeff = cval; }
	int  GetCoeff(void)     { return Coeff; }
	void SetIndex(int ival) { Index = ival; }
	int  GetIndex(void)     { return Index; }
	virtual TDatValue * GetCopy() // изготовить копию
	{
		TDatValue * temp = new TMonom(Coeff, Index);
		return temp;
	}
	TMonom& operator=(const TMonom &tm) {
		Coeff = tm.Coeff; Index = tm.Index;
		return *this;
	}
	int operator==(const TMonom &tm) {
		return (Coeff == tm.Coeff) && (Index == tm.Index);
	}
	int operator<(const TMonom &tm) {
		return Index<tm.Index;
	}
	friend class TPolinom;
};
#endif
```

 __Тесты для класса `TMonom`:__
 
```c++
#include <gtest/gtest.h>
#include "Monom/Monom.h"

TEST(TMonom, can_create_monom)
{
	ASSERT_NO_THROW(TMonom m1);
}

TEST(TMonom, can_create_monom_with_args)
{
	ASSERT_NO_THROW(TMonom m1(-18, 253));
}

TEST(TMonom, can_set_monom_coeff)
{
	TMonom m1;
	ASSERT_NO_THROW(m1.SetCoeff(15));
}

TEST(TMonom, can_get_monom_coeff)
{
	TMonom m1(16, 1);
	EXPECT_EQ(16, m1.GetCoeff());
}

TEST(TMonom, can_set_monom_index)
{
	TMonom m1;
	ASSERT_NO_THROW(m1.SetIndex(18));
}

TEST(TMonom, can_get_monom_index)
{
	TMonom m1(1, 3);
	EXPECT_EQ(3, m1.GetIndex());
}

TEST(TMonom, can_assign_monoms)
{
	TMonom m1(10, 3), m2(-10, 6);
	m2 = m1;
	EXPECT_TRUE((m1.GetCoeff() == m2.GetCoeff())&&(m1.GetIndex() == m2.GetIndex()));
}

TEST(TMonom, can_get_copy)
{
	TMonom m1(210, 567);
	TMonom * m2;
	m2 = (TMonom *)m1.GetCopy();
	EXPECT_TRUE((m1.GetCoeff() == m2->GetCoeff()) && (m1.GetIndex() == m2->GetIndex()));
}

TEST(TMonom, equal_monoms_are_equal)
{
	TMonom m1(10, 91), m2(10, 91);
	EXPECT_TRUE(m1 == m2);
}

TEST(TMonom, monom_with_less_index_is_less)
{
	TMonom m1(10, 91), m2(4, 101);
	EXPECT_TRUE(m1 < m2);
}
```

__Подтверждение успешного прохождения тестов для класса TMonom:__

![gTest.jpg](http://i.imgur.com/Zqlhwfp.jpg "gTest.jpg")

  __Реализация класса `TRootLink`:__

```c++
// RootLink.h
// модуль базового класса для звеньев(элементов) списка

#ifndef _ROOTLINK_H_
#define _ROOTLINK_H_

#include "DatValue/DatValue.h"

class TRootLink;
typedef TRootLink *PTRootLink;

class TRootLink {
protected:
	PTRootLink  pNext;  // указатель на следующее звено
public:
	TRootLink(PTRootLink pN = nullptr) { pNext = pN; }
	PTRootLink  GetNextLink() { return  pNext; }
	void SetNextLink(PTRootLink  pLink) { pNext = pLink; }
	void InsNextLink(PTRootLink  pLink) { // вставка звена
		PTRootLink p = pNext;  pNext = pLink;
		if (pLink != nullptr) pLink->pNext = p;
	}
	virtual void       SetDatValue(PTDatValue pVal) = 0;
	virtual PTDatValue GetDatValue() = 0;

	friend class TDatList;
};
#endif
```
 __Реализация класса `TDatLink`:__

```c++
// DatLink.h
// модуль класса для звеньев(элементов) списка с указателем на объект - значение

#ifndef _DATLINK_H_
#define _DATLINK_H_

#include "RootLink/RootLink.h"

class TDatLink;
typedef TDatLink *PTDatLink;

class TDatLink : public TRootLink {
protected:
	PTDatValue pValue;  // указатель на объект-значение
public:
	TDatLink(PTDatValue pVal = nullptr, PTRootLink pN = nullptr) : TRootLink(pN) 
	{
		pValue = pVal;
	}
	void       SetDatValue(PTDatValue pVal) { pValue = pVal; }
	PTDatValue GetDatValue() { return  pValue; }
	PTDatLink  GetNextDatLink() { return  (PTDatLink)pNext; }
	friend class TDatList;
};
#endif
}
```

 __Реализация класса `TDatList`:__
 
```c++
// DatList.h
// модуль класса линейных списков

#ifndef _DATLIST_H_
#define _DATLIST_H_

#include "DataCom/tdatacom.h"
#include "DatLink/DatLink.h"

#define ListOK 0 // ошибок нет
#define ListEmpty -101 // список пуст
#define ListNoMem -102 // нет памяти
#define ListNoPos -103 // ошибочное положение текущего указателя

enum TLinkPos {FIRST, CURRENT, LAST};

class DatList;
typedef TDatList *PTDatList;

class TDatList : public TDataCom{
protected:
	PTDatLink pFirst;    // первое звено
	PTDatLink pLast;     // последнее звено
	PTDatLink pCurrLink; // текущее звено
	PTDatLink pPrevLink; // звено перед текущим
	PTDatLink pStop;     // значение указателя, означающего конец списка 
	int CurrPos;         // номер текущего звена (нумерация от 0)
	int ListLen;         // количество звеньев в списке
protected:  // методы
	PTDatLink GetLink(PTDatValue pVal = nullptr, PTDatLink pLink = nullptr);
	void      DelLink(PTDatLink pLink);   // удаление звена
public:
	TDatList();
	~TDatList() { DelList(); }

	// доступ
	PTDatValue GetDatValue(TLinkPos mode = CURRENT) const; // значение
	virtual int IsEmpty()  const { return pFirst == pStop; } // проверка пустоты
	int GetListLength()    const { return ListLen; }       // количество звеньев

	// навигация
	int SetCurrentPos(int pos);          // установить текущее звено
	int GetCurrentPos(void) const;       // получить номер тек. звена
	virtual int Reset(void);             // установить текущим первое звено
	virtual bool IsListEnded(void) const; // список завершен ?
	int GoNext(void);                    // сдвиг вправо текущего звена
										 // (=1 после применения GoNext для последнего звена списка)

	// вставка звеньев
	virtual void InsFirst(PTDatValue pVal = nullptr); // перед первым
	virtual void InsLast(PTDatValue pVal = nullptr); // вставить последним 
	virtual void InsCurrent(PTDatValue pVal = nullptr); // перед текущим 

	// удаление звеньев
	virtual void DelFirst(void);    // удалить первое звено 
	virtual void DelCurrent(void);    // удалить текущее звено 
	virtual void DelList(void);    // удалить весь список
};

#endif

// DatList.cpp
// модуль класса линейных списков

#include "DatList/DatList.h"

TDatList::TDatList()
{
	pFirst = pLast = pStop = nullptr;
	ListLen = 0;
}

		/*-------------------------------------------*/

PTDatLink TDatList::GetLink(PTDatValue pVal, PTDatLink pLink)
{
	PTDatLink temp = new TDatLink(pVal, pLink); // выделение звена
	if (temp == nullptr)
		SetRetCode(ListNoMem);
	else
		SetRetCode(ListOK);
	return temp;
}

		/*-------------------------------------------*/

void TDatList::DelLink(PTDatLink pLink) // удаление звена
{
	if (pLink != nullptr)
	{
		if (pLink->pValue != nullptr)
			delete pLink->pValue;
		delete pLink;
	}
	SetRetCode(ListOK);
}

		/*-------------------------------------------*/

PTDatValue TDatList::GetDatValue(TLinkPos mode) const //получить данные звена
{
	PTDatLink temp;
	switch(mode)
	{
		case FIRST: temp = pFirst; break;
		case LAST: temp = pLast; break;
		default: temp = pCurrLink;  break;
	}
	return (temp == nullptr) ? nullptr : temp->pValue;
}

		/*-------------------------------------------*/

// методы навигаии

int TDatList::SetCurrentPos(int pos) // установить текущее звено
{
	Reset(); // возвращаемся в начало списка
	for (int i = 0; i < pos; i++, GoNext())
		SetRetCode(ListOK);
	return RetCode;
}

		/*-------------------------------------------*/

int TDatList::GetCurrentPos(void) const // получить номер текущего звена
{
	return CurrPos;
}

		/*-------------------------------------------*/

int TDatList::Reset(void) // установить текущим первое звено
{
	pPrevLink = pStop;
	if (IsEmpty())
	{
		pCurrLink = pStop;
		CurrPos = -1;
		SetRetCode(ListEmpty);
	}
	else
	{
		pCurrLink = pFirst;
		CurrPos = 0;
		SetRetCode(ListOK);
	}
	return RetCode;
}

		/*-------------------------------------------*/

int TDatList::GoNext(void) // сдвиг текущего звена вправо
{
	if (pCurrLink == pStop)
		SetRetCode(ListNoPos); // следующее звено отсутствует
	else
	{
		SetRetCode(ListOK);
		pPrevLink = pCurrLink; 
		pCurrLink = pCurrLink->GetNextDatLink();
		CurrPos++;
	}
	return RetCode;
}

		/*-------------------------------------------*/

bool TDatList::IsListEnded(void) const // список завершен?
{
	// (== true после применения GoNext для последнего звена списка)
	return pCurrLink == pStop;
}

		/*-------------------------------------------*/

// методы вставки звеньев

void TDatList::InsFirst(PTDatValue pVal) // вставить перед первым
{
	PTDatLink temp = GetLink(pVal, pFirst);
	if (temp == nullptr)
		SetRetCode(ListNoMem);
	else
	{
		pFirst = temp; ListLen++;
		// проверка пустоты списка перед вставкой
		if (ListLen == 1)
		{
			pLast = temp;
			Reset();
		}
		// коррекция текущей позиции - отличие обработки для начала списка
		else
		{
			if (CurrPos == 0)
				pCurrLink = temp;
			else
				CurrPos++;
		}
		SetRetCode(ListOK);
	}
}

		/*-------------------------------------------*/

void TDatList::InsLast(PTDatValue pVal) // вставить последним
{
	PTDatLink temp = GetLink(pVal, pStop);
	if (temp == nullptr)
		SetRetCode(ListNoMem);
	else
	{
		if (pLast != nullptr)
			pLast->SetNextLink(temp);
		pLast = temp; ListLen++;
		// проверка пустоты списка перед вставкой
		if (ListLen == 1)
		{
			pFirst = temp;
			Reset();
		}
		// корректировка текущей позиции - отличие при CurrLink за концом списка
		if (IsListEnded())
			pCurrLink = temp;
		SetRetCode(ListOK);
	}
}

		/*-------------------------------------------*/

void TDatList::InsCurrent(PTDatValue pVal) // вставить перед текущим
{
	if (IsEmpty() || (pCurrLink == pFirst))
		InsFirst(pVal);
	else if (IsListEnded())
		InsLast(pVal);
	else if (pPrevLink == pStop)
		SetRetCode(ListNoMem);
	else
	{
		PTDatLink temp = GetLink(pVal, pCurrLink);
		if (temp == nullptr)
			SetRetCode(ListNoMem);
		else
		{
			pCurrLink = temp; pPrevLink->SetNextLink(temp);
			ListLen++; SetRetCode(ListOK);
		}
	}
}

// методы удаления звеньев

void TDatList::DelFirst(void) // удалить первое звено
{
	if (IsEmpty())
		SetRetCode(ListEmpty);
	else
	{
		PTDatLink temp = pFirst;
		pFirst = pFirst->GetNextDatLink();
		DelLink(temp); ListLen--;
		if (IsEmpty())
		{
			pLast = pStop;
			Reset();
		}
		// корректировка текущей позиции - отличие обработки для начала списка
		else if (CurrPos == 0)
			pCurrLink = pFirst;
		else if (CurrPos == 1)
			pPrevLink = pStop;
		if (CurrPos > 0)
			CurrPos--;
		SetRetCode(ListOK);
	}
}

		/*-------------------------------------------*/

void TDatList::DelCurrent(void) // удалить текущее звено
{
	if (pCurrLink == pStop)
		SetRetCode(ListNoPos);
	else if (pCurrLink == pFirst)
		DelFirst();
	else
	{
		PTDatLink temp = pCurrLink;
		pCurrLink = pCurrLink->GetNextDatLink();
		pPrevLink->SetNextLink(pCurrLink);
		DelLink(temp); ListLen--;
		// обработка ситуации удаления последнего звена
		if (pCurrLink == pLast)
		{
			pLast = pPrevLink;
			pCurrLink = pStop;
		}
		SetRetCode(ListOK);
	}
}

		/*-------------------------------------------*/

void TDatList::DelList(void) // удалить весь список
{
	while (!IsEmpty())
		DelFirst();
	pFirst = pLast = pPrevLink = pCurrLink = pStop;
	CurrPos = -1;
}

```

 __Тесты для класса `TDatList`:__
 
```c++
#include <gtest/gtest.h>
#include "DatList/DatList.h"
#include "DatList/DatList.cpp"
#include "Monom/Monom.h"

TEST(TDatList, can_create_list)
{
	ASSERT_NO_THROW(TDatList l1);
}

TEST(TDatList, new_list_is_empty)
{
	TDatList l1;
	ASSERT_TRUE(l1.IsEmpty());
}

TEST(TDatList, can_put_link_in_list)
{
	TDatList l1;
	PTMonom m1;
	m1 = new TMonom(4, 123);
	ASSERT_NO_THROW(l1.InsLast(m1));
}

TEST(TDatList, list_with_links_is_not_empty)
{
	TDatList l1;
	PTMonom m1, m2;
	m1 = new TMonom(4, 123);
	m2 = new TMonom(8, 456);
	l1.InsLast(m1);
	l1.InsLast(m2);
	ASSERT_TRUE((!l1.IsEmpty()) && (l1.GetListLength() == 2));
}

TEST(TDatList, can_get_current_position)
{
	TDatList l1;
	PTMonom pVal;
	for (int i = 1; i < 6; i++)
	{
		pVal = new TMonom(i, i * 100 + i * 10 + i);
		l1.InsLast(pVal);
	}
	EXPECT_EQ(l1.GetCurrentPos(), 0);
}

TEST(TDatList, can_set_current_position)
{
	TDatList l1;
	PTMonom pVal;
	for (int i = 1; i < 6; i++)
	{
		pVal = new TMonom(i, i * 100 + i * 10 + i);
		l1.InsLast(pVal);
	}
	l1.SetCurrentPos(4);
	EXPECT_EQ(l1.GetCurrentPos(), 4);
}

TEST(TDatList, can_go_next_link)
{
	TDatList l1;
	PTMonom pVal;
	for (int i = 1; i < 6; i++)
	{
		pVal = new TMonom(i, i * 100 + i * 10 + i);
		l1.InsLast(pVal);
	}
	l1.SetCurrentPos(4);
	l1.GoNext();
	EXPECT_EQ(l1.GetCurrentPos(), 5);
}

TEST(TDatList, can_reset_position)
{
	TDatList l1;
	PTMonom pVal;
	for (int i = 1; i < 6; i++)
	{
		pVal = new TMonom(i, i * 100 + i * 10 + i);
		l1.InsLast(pVal);
	}
	l1.SetCurrentPos(4);
	l1.Reset();
	EXPECT_EQ(l1.GetCurrentPos(), 0);
}

TEST(TDatList, ended_list_is_ended)
{
	TDatList l1;
	PTMonom pVal;
	for (int i = 1; i < 6; i++)
	{
		pVal = new TMonom(i, i * 100 + i * 10 + i);
		l1.InsLast(pVal);
	}
	l1.SetCurrentPos(4);
	l1.GoNext();
	EXPECT_TRUE(l1.IsListEnded());
}

TEST(TDatList, can_get_link_from_list)
{
	TDatList l1;
	PTMonom pVal;
	for (int i = 1; i < 6; i++)
	{
		pVal = new TMonom(i, i * 100 + i * 10 + i);
		l1.InsLast(pVal);
	}
	l1.SetCurrentPos(4);
	pVal = (PTMonom)l1.GetDatValue();
	EXPECT_EQ(pVal->GetIndex() + pVal->GetCoeff(), 560);
}

TEST(TDatList, can_put_link_before_the_first)
{
	TDatList l1;
	PTMonom pVal, temp;
	for (int i = 1; i < 6; i++)
	{
		pVal = new TMonom(i, i * 100 + i * 10 + i);
		l1.InsLast(pVal);
	}
	pVal = new TMonom(6, 666);
	l1.InsFirst(pVal);
	temp = (PTMonom)l1.GetDatValue();
	EXPECT_EQ(temp->GetIndex() + temp->GetCoeff(), 672);
}

TEST(TDatList, can_put_link_after_the_last)
{
	TDatList l1;
	PTMonom pVal, temp;
	for (int i = 0; i < 5; i++)
	{
		pVal = new TMonom(i, i * 100 + i * 10 + i);
		l1.InsLast(pVal);
	}
	pVal = new TMonom(6, 666);
	l1.InsLast(pVal);
	l1.SetCurrentPos(5);
	temp = (PTMonom)l1.GetDatValue();
	EXPECT_EQ(temp->GetIndex() + temp->GetCoeff(), 672);
}

TEST(TDatList, can_put_link_before_the_current)
{
	TDatList l1;
	PTMonom pVal, temp;
	for (int i = 0; i < 5; i++)
	{
		pVal = new TMonom(i, i * 100 + i * 10 + i);
		l1.InsLast(pVal);
	}
	l1.SetCurrentPos(2);
	pVal = new TMonom(6, 666);
	l1.InsCurrent(pVal);
	temp = (PTMonom)l1.GetDatValue();
	EXPECT_EQ(temp->GetIndex() + temp->GetCoeff(), 672);
}

TEST(TDatList, can_delete_first_link)
{
	TDatList l1;
	PTMonom pVal, temp;
	for (int i = 0; i < 5; i++)
	{
		pVal = new TMonom(i, i * 100 + i * 10 + i);
		l1.InsLast(pVal);
	}
	temp = (PTMonom)l1.GetDatValue();
	l1.DelFirst();
	pVal = (PTMonom)l1.GetDatValue();
	EXPECT_TRUE((temp->GetIndex() + temp->GetCoeff() != pVal->GetIndex() + pVal->GetCoeff()) && (l1.GetListLength() == 4));
}

TEST(TDatList, can_delete_current_link)
{
	TDatList l1;
	PTMonom pVal;
	for (int i = 0; i < 5; i++)
	{
		pVal = new TMonom(i, i * 100 + i * 10 + i);
		l1.InsLast(pVal);
	}
	l1.SetCurrentPos(3);
	l1.DelCurrent();
	l1.SetCurrentPos(3);
	pVal = (PTMonom)l1.GetDatValue();
	EXPECT_TRUE((pVal->GetIndex() + pVal->GetCoeff() == 448) && (l1.GetListLength() == 4));
}

TEST(TDatList, can_delete_list)
{
	TDatList l1;
	PTMonom pVal;
	for (int i = 0; i < 5; i++)
	{
		pVal = new TMonom(i, i * 100 + i * 10 + i);
		l1.InsLast(pVal);
	}
	l1.DelList();
	EXPECT_TRUE((l1.IsEmpty()) && (l1.GetListLength() == 0) && (l1.GetDatValue() == nullptr));
}
```

__Подтверждение успешного прохождения тестов для класса TDatList:__

![gTest.jpg](http://i.imgur.com/72Md5rq.jpg "gTest.jpg")

 __Реализация класса `THeadRing`:__
 
```c++
// HeadRing.h
// модуль класса циклических списков с заголовком

#ifndef _HEADRING_H_
#define _HEADRING_H_

#include "DatList/DatList.h"

class THeadRing : public TDatList{
protected:
	PTDatLink pHead;     // заголовок, pFirst - будет следующим за pHead
public:
	THeadRing();
	~THeadRing();
	// вставка звеньев
	virtual void InsFirst(PTDatValue pVal = nullptr); // после заголовка
	// удаление звеньев
	virtual void DelFirst(void);                 // удалить первое звено
};
#endif

// HeadRing.cpp
// модуль класса циклических списков с заголовком

#include "HeadRing/HeadRing.h"

THeadRing::THeadRing() : TDatList()
{
	InsLast();
	pHead = pFirst;
	ListLen = 0;
	pStop = pHead;
	Reset();
	pFirst->SetNextLink(pFirst);
}

		/*-------------------------------------------*/

THeadRing::~THeadRing()
{
	TDatList::~TDatList();
	DelLink(pHead);
	pHead = nullptr;
}

		/*-------------------------------------------*/

void THeadRing::InsFirst(PTDatValue pVal) // вставить после заголовка
{
	TDatList::InsFirst(pVal);
	if (RetCode == DataOK)
		pHead->SetNextLink(pFirst);
}

		/*-------------------------------------------*/

void THeadRing::DelFirst(void) // удалить первое звено
{
	TDatList::DelFirst();
	pHead->SetNextLink(pFirst);
}
```

 __Тесты для класса `THeadRing`:__
 
```c++
#include <gtest/gtest.h>
#include "HeadRing/HeadRing.h"
#include "HeadRing/HeadRing.cpp"
#include "Monom/Monom.h"

TEST(THeadRing, can_create_headring_list)
{
	ASSERT_NO_THROW(THeadRing l1);
}

TEST(THeadRing, new_headring_list_is_empty)
{
	THeadRing l1;
	ASSERT_TRUE(l1.IsEmpty());
}

TEST(THeadRing, can_put_link_before_the_first)
{
	THeadRing l1;
	PTMonom pVal, temp;
	for (int i = 0; i < 3; i++)
	{
		pVal = new TMonom(i, i * 100 + i * 10 + i);
		l1.InsLast(pVal);
	}
	pVal = new TMonom(3, 333);
	l1.InsFirst(pVal);
	temp = (PTMonom)l1.GetDatValue();
	EXPECT_EQ(temp->GetIndex() + temp->GetCoeff(), 336);
}

TEST(THeadRing, can_delete_first_link)
{
	THeadRing l1;
	PTMonom pVal, temp;
	for (int i = 1; i < 5; i++)
	{
		pVal = new TMonom(i, i * 100 + i * 10 + i);
		l1.InsLast(pVal);
	}
	l1.DelFirst();
	l1.SetCurrentPos(0);
	temp = (PTMonom)l1.GetDatValue();
	EXPECT_TRUE((temp->GetIndex() + temp->GetCoeff() == 224) && (l1.GetListLength() == 3));
}
```

__Подтверждение успешного прохождения тестов для класса THeadRing:__

![gTest.jpg](http://i.imgur.com/DkQ7skE.jpg "gTest.jpg")

 __Реализация класса `TPolinom`:__

```c++
// Polinom.h
// модуль класса полиномов

#ifndef _POLINOM_H_
#define _POLINOM_H_

#include <iostream>
#include "HeadRing/HeadRing.h"
#include "Monom/Monom.h"

using namespace std;

class TPolinom : public THeadRing {
private:
	long GetPow(long val, int power);
public:
	TPolinom(int monoms[][2] = nullptr, int km = 0); // конструктор полинома 
													 // из массива «коэффициент-индекс»
	TPolinom(TPolinom &q); // конструктор копирования
	PTMonom  GetMonom()  { return (PTMonom)GetDatValue(); }
	TPolinom & operator+(TPolinom &q); // сложение полиномов
	long CalculatePoly(int x, int y, int z); // вычислить значение полинома 
	bool operator==(TPolinom &q); // сравнение полиномов
	bool operator!=(TPolinom &q); // сравнение полиномов
	TPolinom & operator=(TPolinom &q); // присваивание
	friend ostream& operator<<(ostream &os, TPolinom &q); // печать полинома
	friend istream& operator>>(istream &is, TPolinom &q); // ввод полинома
};
#endif

// Polinom.cpp
// модуль класса полиномов

#include "Polinom/Polinom.h"

TPolinom::TPolinom(int monoms[][2], int km)
{
	PTMonom pMonom = new TMonom(0, -1);
	pHead->SetDatValue(pMonom);
	for (int i = 0; i < km; i++)
	{
		pMonom = new TMonom(monoms[i][0], monoms[i][1]);
		InsLast(pMonom);
	}
}

		/*-------------------------------------------*/

TPolinom & TPolinom::operator+(TPolinom &q) // сложение полиномов
{
	PTMonom pm, qm, tm;
	Reset(); q.Reset();
	while (true)
	{
		pm = GetMonom(); qm = q.GetMonom();
		if (pm->Index < qm->Index)
		{
			// степени монома pm меньше степеней монома qm 
			// >> добавление монома qm  в полином p
			tm = new TMonom(qm->Coeff, qm->Index);
			InsCurrent(tm); q.GoNext();
		}
		else if (pm->Index > qm->Index)
			GoNext();
		else // индексы мономов равны
		{
			if (pm->Index == -1) // звенья не должны быть заголовочными
				break; 
			pm->Coeff += qm->Coeff;
			if (pm->Coeff != 0)
			{
				GoNext(); q.GoNext();
			}
			else // удаление монома с нулевым коэффициентом
			{
				DelCurrent(); q.GoNext();
			}
		}
	}
	return *this;
}

		/*-------------------------------------------*/

long TPolinom::GetPow(long val, int power)
{
	long result = 1;
	for (int i = 0; i < power; i++)
		result *= val;
	return result;
}

	/*-------------------------------------------*/

long TPolinom::CalculatePoly(int x, int y, int z) // вычислить значение полинома 
{
	long result = 0;
	if (GetListLength() != 0)
	{
		PTMonom pMonom;
		int cf, index;
		for (Reset(); !IsListEnded(); GoNext())
		{
			pMonom = GetMonom();
			cf = pMonom->GetCoeff();
			index = pMonom->GetIndex();
			result += cf * GetPow(x, index / 100) * GetPow(y, (index % 100) / 10) * GetPow(z, index % 10);
		}
	}
	return result;
}

		/*-------------------------------------------*/

bool TPolinom::operator==(TPolinom &q) // сравнение полиномов
{
	if (GetListLength() != q.GetListLength())
		return false;
	else
	{
		PTMonom pMon, qMon;
		Reset(); q.Reset();
		while (!IsListEnded())
		{
			pMon = GetMonom();
			qMon = q.GetMonom();
			if (*pMon == *qMon)
			{
				GoNext(); q.GoNext();
			}
			else
				return false;
		}
		return true;
	}
}

		/*-------------------------------------------*/

bool TPolinom::operator!=(TPolinom &q) // сравнение полиномов
{
	return !(*this == q);
}
		/*-------------------------------------------*/

TPolinom::TPolinom(TPolinom &q) // конструктор копирования
{
	PTMonom pMonom = new TMonom(0, -1);
	pHead->SetDatValue(pMonom);
	for  ( q.Reset();  !q.IsListEnded(); q.GoNext())
	{
		pMonom = q.GetMonom();
		InsLast(pMonom->GetCopy());
	}
	q.Reset();
}

		/*-------------------------------------------*/

TPolinom & TPolinom::operator=(TPolinom &q) // присваивание
{
	if (this != &q)
	{
		PTMonom pMonom;
		DelList();
		for (q.Reset(); !q.IsListEnded(); q.GoNext())
		{
			pMonom = q.GetMonom();
			InsLast(pMonom->GetCopy());
		}
		q.Reset();
	}
	return *this;
}

		/*-------------------------------------------*/

ostream& operator<<(ostream &os, TPolinom &q)
{
	PTMonom pMonom;
	int cf, x, y, z;
	if (q.GetListLength() > 0)
	{
		for (q.Reset(); !q.IsListEnded(); q.GoNext())
		{
			pMonom = q.GetMonom();
			cf = pMonom->GetCoeff();
			x = pMonom->GetIndex();
			y = (x % 100) / 10;
			z = x % 10;
			x = x / 100;
			if (cf != 0)
			{
				if ((cf > 0)&&(q.GetCurrentPos() != 0))
					os << "+" << cf;
				else
					os << cf;
				if (x > 0)
					os << "x^" << x;
				if (y > 0)
					os << "y^" << y;
				if (z > 0)
					os << "z^" << z;
				os << " ";
			}
		}
	}
	else
		os << 0;
	q.Reset();
	return os;
}

		/*-------------------------------------------*/

istream& operator>>(istream &is, TPolinom &q)
{
	if (q.GetListLength() > 0)
		q.DelList();
	char polystr[256];
	char mnstr[64];
	PTMonom tempMonom;
	int pos = 0, i, cf, x , y ,z;
	is.getline(polystr, 256);
	q.Reset();
	while (polystr[pos])
	{
		i = 0;
		while ((polystr[pos] != ' ') && (polystr[pos]))
		{	
			if (polystr[pos] != '+')
			{
				mnstr[i] = polystr[pos];
				i++;
			}
			pos++;
		}
		mnstr[i] = 0;
		i = 0;
		if (mnstr[0] == '-')
			i++;
		cf = x = y = z = 0; 
		while ((mnstr[i] != 'x') && (mnstr[i] != 'y') && (mnstr[i] != 'z') && (mnstr[i] != 0))
		{
			cf = cf * 10 + (mnstr[i] - '0');
			i++;
		}
		if (mnstr[0] == '-')
			cf *= -1;
		if (mnstr[i] == 'x')
		{
			i += 2;
			x = mnstr[i] - '0';
		}
		while ((mnstr[i] != 'y') && (mnstr[i] != 'z') && (mnstr[i] != 0))
			i++;
		if (mnstr[i] == 'y')
		{
			i += 2;
			y = mnstr[i] - '0';
		}
		while ((mnstr[i] != 'z') && (mnstr[i] != 0))
			i++;
		if (mnstr[i] == 'z')
		{
			i += 2;
			z = mnstr[i] - '0';
		}
		tempMonom = new TMonom(cf, x * 100 + y * 10 + z);
		q.InsLast(tempMonom);
		if (polystr[pos] != 0)
			pos++;
		mnstr[0] = 0;
	}
	q.Reset();
	return is;
}
```

 __Тесты для класса `TPolinom`:__
 
```c++
#include <gtest/gtest.h>
#include "Polinom/Polinom.h"
#include "Polinom/Polinom.cpp"

TEST(TPolinom, can_create_polynomial)
{
	int monoms[][2] = { { 234, 234 }, { 230, 230 }, { 204, 204 }, { 24, 24 } };
	ASSERT_NO_THROW(TPolinom poly(monoms, 4));
}

TEST(TPolinom, can_compare_polynominals)
{
	int monoms1[][2] = { { 234, 234 }, { 230, 230 }, { 204, 204 }};
	int monoms2[][2] = { { 24, 234 }, { 230, 230 }};
	TPolinom poly1(monoms1, 3), poly2(monoms1, 3), poly3(monoms2, 2);
	ASSERT_TRUE((poly1 == poly2) && (poly1 != poly3));
}

TEST(TPolinom, can_create_polinominal_with_copy_constr)
{
	int monoms[][2] = { { 24, 234 }, { 230, 230 } };
	TPolinom poly1(monoms, 2);
	TPolinom poly2(poly1);
	ASSERT_TRUE(poly1 == poly2);
}

TEST(TPolinom, can_assign_polynominal)
{
	int monoms1[][2] = { { 234, 234 }, { 230, 230 }, { 204, 204 } };
	int monoms2[][2] = { { 24, 234 }, { 230, 230 } };
	TPolinom poly1(monoms1, 3), poly2(monoms2, 2);
	poly2 = poly1;
	ASSERT_TRUE(poly1 == poly2);
}

TEST(TPolinom, copied_polynominal_has_its_own_mem)
{
	int monoms1[][2] = { { 234, 234 }, { 230, 230 }, { 204, 204 } };
	int monoms2[][2] = { { 24, 234 }, { 230, 230 } };
	TPolinom poly1(monoms1, 3), poly2(monoms2, 2), poly3(poly2);;
	poly1 = poly2;
	poly2.DelFirst();
	ASSERT_FALSE((poly1 == poly2) || (poly2 == poly3));
}

TEST(TPolinom, can_sum_polynomial)
{
	int monoms1[][2] = { { 234, 234 }, { 230, 230 }, { 24, 24 }, { 1, 12 } };
	int monoms2[][2] = { { -234, 234 }, { -24, 24 } };
	int monoms3[][2] = { { 230, 230 }, {1, 12} };
	TPolinom poly1(monoms1, 4), poly2(monoms2, 2), respoly(monoms3, 2);
	poly1 = poly1 + poly2;
	ASSERT_TRUE(poly1 == respoly);
}

TEST(TPolinom, can_calculate_polynominal)
{
	int monoms[][2] = { { 4, 123 }, { -2, 110 }, { 4, 90 } };
	TPolinom poly1(monoms, 3), poly2;
	EXPECT_EQ(2296, poly1.CalculatePoly(2, 2, 2));
}
```

__Подтверждение успешного прохождения тестов для класса TPolinom:__

![gTest.jpg](http://i.imgur.com/CIPRTWl.jpg "gTest.jpg")

 __Пример использования класса `TPolinom`:__

```c++
// poly_testkit.cpp
// пример испозьвания класса TPolinom

#include "Polinom/Polinom.h"

int main()
{
	setlocale(LC_ALL, "Russian");
	cout << "Тестирование полиномов" << endl;

	TPolinom poly1, poly2, temp;
	int x, y, z;

	cout << "Введите полиномы: " << endl;
	cin >> poly1;
	cin >> poly2;
	if (poly1 == poly2)
		cout << "Полиномы равны" << endl;
	else
		cout << "Полиномы не равны" << endl;
	cout << poly1 << " + " << poly2 << " = " << endl;
	poly1 + poly2;
	cout << poly1 << endl;
	cout << "Вычислить значение суммы при: ";
	cin >> x >> y >> z; cout << endl;
	cout << "Результат: " << poly1.CalculatePoly(x, y, z) << endl;
	return 0;
}
```

__Результат работы тестового приложения:__

![gTest.jpg](http://i.imgur.com/6dqRs4a.jpg "gTest.jpg")

__Подтверждение правильности вычислений:__

![gTest.jpg](http://i.imgur.com/pGggn1v.jpg "gTest.jpg")

## Используемые инструменты

  - Система контроля версий [Git][git].
  - Фреймворк для написания автоматических тестов [Google Test][gtest].
  - Среда разработки Microsoft Visual Studio 2013. 

## Вывод  

Выполнение данной работы помогло мне освежить в памяти основные принципы наследования классов в C++, закрепило знания о фреймворке для написания автоматических тестов [Google Test][gtest].

При выполнении работы возникли некоторые трудности с вводом\выводом полиномов. Реализация методов, предоставляющих данные возможности, могла бы выглядеть более лаконично, если бы были функции ввода\вывода для мономов. Однако и приведенные методы справляются со своей задачей, они могут быть улучшены при дальнейшем развитии класса.

<!-- LINKS -->

[git]:         https://git-scm.com/book/ru/v2
[gtest]:       https://github.com/google/googletest
[sieve]:       http://habrahabr.ru/post/91112
[git-guide]:   https://bitbucket.org/ashtan/mp2-lab1-set/src/ff6d76c3dcc2a531cefdc17aad5484c9bb8b47c5/docs/part1-git.md?at=master&fileviewer=file-view-default
[gtest-guide]: https://bitbucket.org/ashtan/mp2-lab1-set/src/ff6d76c3dcc2a531cefdc17aad5484c9bb8b47c5/docs/part2-google-test.md?at=master&fileviewer=file-view-default
[upstream]:    https://bitbucket.org/ashtan/mp2-lab1-set 