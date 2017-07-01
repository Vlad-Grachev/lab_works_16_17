# Лабораторная работа №4. Очереди#


## Цели и задачи

Лабораторная работа направлена на практическое освоение динамической структуры данных Очередь. С этой целью в лабораторной работе изучаются различные варианты структуры хранения очереди и разрабатываются методы и программы решения задач с использованием очередей. В качестве области приложений выбрана тема эффективной организации выполнения потока заданий на вычислительных системах.

Очередь характеризуется таким порядком обработки значений, при котором вставка новых элементов производится в конец очереди, а извлечение – из начала. Подобная организация данных широко встречается в различных приложениях. В качестве примера использования очереди предлагается задача разработки системы имитации однопроцессорной ЭВМ. Рассматриваемая в рамках лабораторной работы схема имитации является одной из наиболее простых моделей обслуживания заданий в вычислительной системе и обеспечивает тем самым лишь начальное ознакомление с проблемами моделирования и анализа эффективности функционирования реальных вычислительных систем.

Результатом выполнения работы является следующий набор файлов:

- **TStack.h, TStack.cpp** – модуль с классом, реализующим операции над стеком (был взят из предыдущей лабороторной работы);
- **TQueue.h, TQueue.cpp** – модуль с классом, реализующим операции над очередью;
- **TProc.h, TProc.cpp** – модуль с классом, реализующим процессор;
- **TJobStream.h, TJobStream.cpp** – модуль с классом, реализующим поток задач;
- **QueueTestkit.cpp** – модуль программы тестирования.
 
 __Реализация класса `TQueue`:__

```c++
// tqueue.h
// Динамические структуры данных - очередь

#ifndef _TQUEUE_H_
#define _TQUEUE_H_

#include "tdatstack.h"

class TQueue : public TStack
{
protected:
	int Li; // индекс первого элемента структуры
	virtual int GetNextIndex(int Index);
public:
	TQueue(int Size = DefMemSize) : TStack(Size)
	{
		Li = -1;
	}
	virtual void Put(const TData &Val); // добавить элемент в очередь
	virtual TData Get(void); // взять элемент из очереди
};
#endif

// tqueue.cpp
// Динамические структуры данных -  очередь

#include <stdio.h>
#include "tqueue.h"

void TQueue::Put(const TData &Val) //добавить элемент в очередь
{
	if (pMem == NULL)
		SetRetCode(DataNoMem);
	else
		if (IsFull())
			SetRetCode(DataFull);
	else
	{
		Hi = GetNextIndex(Hi);
		pMem[Hi] = Val;
		DataCount++;
	}
}

/*-------------------------------------------*/

TData TQueue::Get(void) // взять элемент из очереди
{
	TData temp = -1;
	if (pMem == NULL)
		SetRetCode(DataNoMem);
	else
	if (IsEmpty())
		SetRetCode(DataEmpty);
	else
	{
		Li = GetNextIndex(Li);
		temp = pMem[Li];
		DataCount--;
	}
	return temp;
}

/*-------------------------------------------*/

int TQueue::GetNextIndex(int index) // получить следующее значение индекса
{
	if (index + 1 == MemSize)
		return 0;
	else
		return ++index;
}
```

__Подтверждение успешного прохождения тестов для класса TQueue:__

![gTest.jpg](http://i.imgur.com/2vkze8X.jpg "gTest.jpg")

 __Реализация класса `TProc`:__

```c++
// tproc.h
// Класс, имитирующий процессор

#ifndef _TPROC_H_
#define _TPROC_H_

#include <ctime>
#include <stdlib.h>
#include "tqueue.h"

#define defpipe_size 10
#define defperf 50

class TProc
{
	private:
		int s = 10;
		TQueue pipeline; // очередь, хранящая задания
		int perf; // вероятность выполнения задания на текущем такте
	public:
		TProc(int psize = defpipe_size, int pf = defperf) : pipeline(psize), perf(pf) { srand(time(0)); };
		bool CanTakeTask(); // наличие места для очередного задания
		bool IsBusy(); // наличие задания, находящегося на выполнении
		bool NewTask(long task_id); // отправить задание на вылонение
		bool IsTaskDone(); // обработка задания
};
#endif

// tproc.cpp
// класс, имитирующий процессор

#include "tproc.h"

bool TProc::CanTakeTask() // наличие места для очередного задания
{
	if (!pipeline.IsFull())
		return true;
	else 
		return false;
}

/*-------------------------------------------*/


bool TProc::IsBusy() // наличие задания, находящегося на выполнении
{
	if (!pipeline.IsEmpty())
		return true;
	else
		return false;
}

/*-------------------------------------------*/

bool TProc :: NewTask(long task_id) // отправить задание на вылонение
{
	if (CanTakeTask())
	{
		pipeline.Put(task_id);
		return true;
	}
	else
		return false;
}

/*-------------------------------------------*/

bool TProc::IsTaskDone() // обработка задания
{
	int chance = rand() % 100;
	if (chance < perf)
	{
		pipeline.Get();
		return true;
	}
	else
		return false;
}
```
 __Реализация класса `TJobStream`:__
 
```c++
// tjobstream.h
// класс, имитирующий поток заданий

#ifndef _TJOBSTRM_H_
#define _TJOBSTRM_H_

#include <iostream>
#include <locale>
#include "tproc.h"

using namespace std;

class TJobStream
{
	private:
		TProc CPunit;
		long tactsNum; // продолжительность работы в тактах
		long taskID; // идентификатор задания / общее число заданий 
		long cmpltd_tasks = 0; // число выполненных заданий
		long idle_tacts = 0; // число тактов простоя
		long deniels = 0; // число отказов в обслуживании
		int intense; // вероятность генерации задания на текущем такте
	public:
		TJobStream(double proc_perf, int pipeline_len, long tactsNum, int intns) : 
			CPunit(pipeline_len, proc_perf), tactsNum(tactsNum), intense(intns) {};
		void StartStream(); // запустить поток
		void PrintReport(); // печать отчета о работе
};
#endif 

// tjobstream.cpp
// класс, имитирующий поток заданий

#include "tjobstream.h"

void TJobStream::StartStream() // запустить поток
{
	taskID = 0; 
	int chance;
	for (long i = 0; i < tactsNum; i++)
	{
		chance = rand() % 100;
		if (chance < intense)
		{
			taskID++;
			if (CPunit.CanTakeTask())
			{
				CPunit.NewTask(taskID);
			}
			else
				deniels++; // отказ в обсуживании 
		}
		if (CPunit.IsBusy()) // Есть ли выполняющиеся задания?
		{
			if (CPunit.IsTaskDone())
				cmpltd_tasks++;
		}
		else
			idle_tacts++; // Общее число тактов работы процессора будет результатом 
	}					  // разности между общим числом тактов и числом тактов простоя
}

/*-------------------------------------------*/

void TJobStream::PrintReport() // печать отчета о работе
{
	setlocale(LC_ALL, "rus");
	if (false)
		cout << "Поток не был запущен" << endl;
	else
	{
		double temp;
		cout << "Количество поступивших в ВС заданий: " << taskID << endl;
		cout << "Количество обработанных заданий: " << cmpltd_tasks << endl;
		cout << "Число отказов в обслуживании: " << deniels << endl;
		temp = (deniels * 100.0) / taskID;
		cout << "Процент отказа в обслуживании: " << temp << "%" << endl;
		temp = (tactsNum - idle_tacts) / cmpltd_tasks;
		cout << "Среднее кол-во тактов на выполнение задания: " << temp << endl;
		cout << "Число тактов простоя процессора: " << idle_tacts << endl;
		temp = (idle_tacts * 100.0) / tactsNum;
		cout << "Процент простоя процессора: " << temp << "%" << endl;
	}
}
```

![change_queuesize.jpg](http://i.imgur.com/jBJzxPQ.jpg "change_queuesize.jpg")

![change_stream_intense.jpg](http://i.imgur.com/gdSqUZt.jpg "change_stream_intense")

![change_cpu_perf.jpg](http://i.imgur.com/YdCRTqi.jpg "change_cpu_perf.jpg")

## Используемые инструменты

  - Система контроля версий [Git][git].
  - Фреймворк для написания автоматических тестов [Google Test][gtest].
  - Среда разработки Microsoft Visual Studio 2013. 

##Вывод  


Выполнение данной работы помогло мне освежить в памяти основные принципы наследования классов в C++, закрепило знания о фреймворке для написания автоматических тестов [Google Test][gtest].

<!-- LINKS -->

[git]:         https://git-scm.com/book/ru/v2
[gtest]:       https://github.com/google/googletest
[sieve]:       http://habrahabr.ru/post/91112
[git-guide]:   https://bitbucket.org/ashtan/mp2-lab1-set/src/ff6d76c3dcc2a531cefdc17aad5484c9bb8b47c5/docs/part1-git.md?at=master&fileviewer=file-view-default
[gtest-guide]: https://bitbucket.org/ashtan/mp2-lab1-set/src/ff6d76c3dcc2a531cefdc17aad5484c9bb8b47c5/docs/part2-google-test.md?at=master&fileviewer=file-view-default
[upstream]:    https://bitbucket.org/ashtan/mp2-lab1-set 

