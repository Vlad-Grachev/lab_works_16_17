# Лабораторная работа №6. Тексты #


## Введение

Обработка текстовой информации на компьютере широко применяется в различных областях человеческой деятельности: образование, наука, документооборот, кадровый и бухгалтерский учет и др. Вне зависимости от назначения текста типовыми операциями обработки являются создание, просмотр, редактирование и сохранение информации. В связи с тем, что объем текстовой информации может являться очень большим, для эффективного выполнения операций с ней необходимо выбрать представление текста, обеспечивающее структурирование и быстрый доступ к различным элементам текста. Так, текст можно представить в виде линейной последовательности страниц, каждая из которых есть линейная последовательность строк, которые  в свою очередь являются линейными последовательностями слов. Такое представление можно осуществлять с любой степенью детализации в зависимости от особенностей прикладной задачи.
В рамках лабораторной работы рассматривается задача разработки учебного редактора текстов, в котором для представления данных используется иерархический связный список. Подобная иерархическая структура представления может применяться при компьютерной реализации математических моделей в виде деревьев и, тем самым, может иметь самое широкое применение в самых различных областях приложений. 

## Цели и задачи

В рамках лабораторной работы была поставлена задача разработки учебного редактора текстов с поддержкой следующих операций:

- выбор текста для редактирования (или создание нового текста);
- демонстрация текста на экране дисплея;
- поддержка средств указания элементов (уровней) текста;
- вставка, удаление и замена строк текста;
- запись подготовленного текста в файл.

При выполнении операций чтения используется стандартный формат, принятый в файловой системе для представления текстовых файлов. 


## Условия и ограничения

При планировании структуры текста в качестве самого нижнего уровня был рассмотрен уровень строк.

**Результатом выполнения работы является следующий набор файлов:**

- DatValue.h – модуль, объявляющий абстрактный класс объектов-значений списка;
- TTextLink.h, TTextLink.cpp – модуль с классом для звена текста;
- TText.h, TText.cpp – модуль с классом, реализующим операции над текстом;
- TextTestKit.cpp – модуль программы тестирования.

__Реализация класса `TTextLink`:__

```c++
// TTextLink.h
// Класс объектов-значений для строк текста

#ifndef _TEXTLINK_H_
#define _TEXTLINK_H_

#include "TDatValue.h"
#include <iostream>
#include <string>

#define TextLineLength 30
#define MemSize 20

using namespace std;

class TText;
class TTextLink;
typedef TTextLink *PTTextLink;
typedef char TStr[TextLineLength];

class TTextMem {
    PTTextLink pFirst;     // указатель на первое звено
    PTTextLink pLast;      // указатель на последнее звено
    PTTextLink pFree;      // указатель на первое свободное звено
    friend class TTextLink;
};
typedef TTextMem *PTTextMem;

class TTextLink : public TDatValue {
protected:
    TStr Str;  // поле для хранения строки текста
    PTTextLink pNext, pDown;  // указатели на тек. уровень и на подуровень
    static TTextMem MemHeader; // система управления памятью
public:
    static void InitMemSystem(int size = MemSize); // инициализация памяти
    static void PrintFreeLink(void);  // печать свободных звеньев
    void * operator new (size_t size); // выделение звена
    void operator delete (void *pM);   // освобождение звена
    static void MemCleaner(TText &txt); // сборка мусора
    TTextLink(TStr s = nullptr, PTTextLink pn = nullptr, PTTextLink pd = nullptr) {
        pNext = pn; pDown = pd;
        if (s != nullptr) strcpy_s(Str, s); else Str[0] = '\0';
    }
    TTextLink(string s) {
        pNext = nullptr; pDown = nullptr;
        strcpy_s(Str, s.c_str());
    }
    ~TTextLink() {}
    int IsAtom() { return pDown == nullptr; } // проверка атомарности звена
    PTTextLink GetNext() { return pNext; }
    PTTextLink GetDown() { return pDown; }
    PTDatValue GetCopy() { return new TTextLink(Str, pNext, pDown); }
protected:
    virtual void Print(ostream &os) { os << Str; }
    friend class TText;
};

#endif

// TTextLink.cpp
// Класс объектов-значений для строк текста

#include "include/TTextLink.h"
#include "include/TText.h"

using namespace std;
TTextMem TTextLink::MemHeader;

void TTextLink::InitMemSystem(int size) { // инициализация памяти
    MemHeader.pFirst = (PTTextLink) new char[sizeof(TTextLink)* size];
    MemHeader.pFree = MemHeader.pFirst;
    MemHeader.pLast = MemHeader.pFirst + (size - 1);
    PTTextLink pLink = MemHeader.pFirst;
    for (int i = 0; i < size - 1; i++, pLink++) {
        pLink->pNext = pLink + 1;
    }
    pLink->pNext = nullptr;
}

        /*-------------------------------------------*/

void TTextLink::PrintFreeLink(void) { // печать свободных звеньев
    PTTextLink pLink = MemHeader.pFree;
    for (; pLink != nullptr; pLink = pLink->pNext)
        cout << pLink->Str << endl;
}

        /*-------------------------------------------*/

void * TTextLink::operator new(size_t size) { // выделение звена
    PTTextLink pLink = MemHeader.pFree;
    if (MemHeader.pFree != nullptr)
        MemHeader.pFree = pLink->pNext;
    return pLink;
}
        /*-------------------------------------------*/

void TTextLink::operator delete(void *pM) { // освобождение звена
    PTTextLink pLink = (PTTextLink)pM;
    pLink->pNext = MemHeader.pFree;
    MemHeader.pFree = pLink;
}

void TTextLink::MemCleaner(TText &txt) { // сборка мусора
    for (txt.Reset(); !txt.IsTextEnded(); txt.GoNext()) {
        txt.SetLine("&&&" + txt.GetLine());
    }

    PTTextLink pLink = MemHeader.pFree;
    for (; pLink != nullptr; pLink->pNext)
        strcpy_s(pLink->Str, "&&&");

    pLink = MemHeader.pFirst;
    for (; pLink <= MemHeader.pLast; pLink++) {
        if (strstr(pLink->Str, "&&&") != nullptr) {
            strcpy_s(pLink->Str, pLink->Str + 3);
        }
        else delete pLink;
    }
}

```

__Реализация класса `TText`:__

```c++
// TText.h
// Тексты - иерархическая структура представления

#ifndef _TTEXT_H_
#define _TTEXT_H_

#include <stack>
#include <string>
#include <fstream>
#include "TDataCom.h"
#include "TTextLink.h"

class TText;
typedef TText *PTText;

class TText : public TDataCom {
protected:
    PTTextLink pFirst;      // указатель корня дерева
    PTTextLink pCurrent;      // указатель текущей строки
    stack< PTTextLink > Path; // стек траектории движения по тексту
                              // (звено pCurrent в стек не включается)
    stack< PTTextLink > St;   // стек для итератора
    PTTextLink GetFirstAtom(PTTextLink pl); // поиск первого атома
    void PrintText(PTTextLink ptl);         // печать текста со звена ptl
    PTTextLink ReadText(ifstream &TxtFile); //чтение текста из файла
    void PrintTextFile(PTTextLink ptl, ofstream& TxtFile); // запись текста в файл
public:
    TText(PTTextLink pl = nullptr);
    ~TText() { pFirst = nullptr; }
    PTText GetCopy();
    // навигация
    int GoFirstLink(void); // переход к первой строке
    int GoDownLink(void);  // переход к следующей строке по Down
    int GoNextLink(void);  // переход к следующей строке по Next
    int GoPrevLink(void);  // переход к предыдущей позиции в тексте
    // доступ
    string GetLine(void);   // чтение текущей строки
    void SetLine(string s); // замена текущей строки 
    // модификация
    void InsDownLine(string s);    // вставка строки в подуровень
    void InsDownSection(string s); // вставка раздела в подуровень
    void InsNextLine(string s);    // вставка строки в том же уровне
    void InsNextSection(string s); // вставка раздела в том же уровне
    void DelDownLine(void);        // удаление строки в подуровне
    void DelDownSection(void);     // удаление раздела в подуровне
    void DelNextLine(void);        // удаление строки в том же уровне
    void DelNextSection(void);     // удаление раздела в том же уровне
    // итератор
    int Reset(void);              // установить на первую запись
    int IsTextEnded(void) const;  // текст завершен?
    int GoNext(void);             // переход к следующей записи
    //работа с файлами
    void Read(string pFileName);  // ввод текста из файла
    void Write(string pFileName); // вывод текста в файл
    //печать
    void Print(void);             // печать текста
};

#endif

// TText.cpp
// Тексты - иерархическая структура представления

#include "include/TText.h"

static int TextLevel; // номер текущего уровня текста

TText::TText(PTTextLink pl) {
    if (pl == nullptr)
        pl = new TTextLink;
    pFirst = pl;
}

        /*-------------------------------------------*/

// навигация
int TText::GoFirstLink(void) { // переход к первой строке
    while (!Path.empty())
        Path.pop();
    pCurrent = pFirst;
    if (pCurrent == nullptr)
        SetRetCode(TextError);
    else
        SetRetCode(TextOK);
    return RetCode;
}

int TText::GoDownLink(void) { //  переход к следующей строке по Down
    SetRetCode(TextNoDown);
    if (pCurrent != nullptr)
        if (pCurrent->pDown != nullptr) {
            Path.push(pCurrent);
            pCurrent = pCurrent->pDown;
            SetRetCode(TextOK);
        }
     return RetCode;
}

int TText::GoNextLink(void) { // переход к следующей строке по Next
    SetRetCode(TextNoNext);
    if (pCurrent != nullptr)
        if (pCurrent->pNext != nullptr) {
            Path.push(pCurrent);
            pCurrent = pCurrent -> pNext;
            SetRetCode(TextOK);
        }
     return RetCode;
}

int TText::GoPrevLink(void) {
    if (Path.empty())
        SetRetCode(TextNoPrev);
    else {
        pCurrent = Path.top();
        Path.pop();
        SetRetCode(TextOK);
    }
    return RetCode;
}

        /*-------------------------------------------*/

// доступ
string TText::GetLine(void) { // чтение текущей строки
    if (pCurrent == nullptr) {
        SetRetCode(TextError);
        return "";
    }
    else {
        return string(pCurrent->Str);
    }
}

void TText::SetLine(string s) { // замена текущей строки
    if (pCurrent == nullptr)
        SetRetCode(TextError);
    else
        strcpy_s(pCurrent->Str, s.c_str());
}

        /*-------------------------------------------*/

// модификация
void TText::InsDownLine(string s) { // вставка строки в подуровень
    if (pCurrent == nullptr)
        SetRetCode(TextError);
    else {
        TStr buf;
        strcpy_s(buf, s.c_str());
        pCurrent->pDown = new TTextLink(buf, pCurrent->pDown, nullptr);
        SetRetCode(TextOK);
    }
}

void TText::InsDownSection(string s) { // вставка раздела в подуровень
    if (pCurrent == nullptr)
        SetRetCode(TextError);
    else {
        TStr buf;
        strcpy_s(buf, s.c_str());
        pCurrent->pDown = new TTextLink(buf, nullptr, pCurrent->pDown);
        SetRetCode(TextOK);
    }
}

void TText::InsNextLine(string s) { // вставка строки в том же уровне
    if (pCurrent == nullptr)
        SetRetCode(TextError);
    else {
        TStr buf;
        strcpy_s(buf, s.c_str());
        pCurrent->pNext = new TTextLink(buf, pCurrent->pNext, nullptr);
        SetRetCode(TextOK);
    }
}

void TText::InsNextSection(string s){ // вставка раздела в том же уровне
    if (pCurrent == nullptr)
        SetRetCode(TextError);
    else {
        TStr buf;
        strcpy_s(buf, s.c_str());
        pCurrent->pNext = new TTextLink(buf, nullptr, pCurrent->pNext);
        SetRetCode(TextOK);
    }
}

void TText::DelDownLine(void) { // удаление строки в подуровне
    SetRetCode(TextOK);
    if (pCurrent == nullptr)
        SetRetCode(TextError);
    else if (pCurrent->pDown == nullptr)
        SetRetCode(TextNoDown);
    else if (pCurrent->pDown->IsAtom())
        pCurrent->pDown = pCurrent->pDown->pNext;
}

void TText::DelDownSection(void) { // удаление раздела в подуровне 
    SetRetCode(TextOK);
    if (pCurrent == nullptr)
        SetRetCode(TextError);
    else if (pCurrent->pDown == nullptr)
        SetRetCode(TextNoDown);
    else {
        pCurrent->pDown = nullptr;
    }
}

void TText::DelNextLine(void) { // удаление строки в том же уровне
    SetRetCode(TextOK);
    if (pCurrent == nullptr)
        SetRetCode(TextError);
    else if (pCurrent->pNext == nullptr)
        SetRetCode(TextNoNext);
    else if (pCurrent->pNext->IsAtom())
        pCurrent->pNext = pCurrent->pNext->pNext;
}

void TText::DelNextSection(void) { // удаление раздела в том же уровне
    SetRetCode(TextOK);
    if (pCurrent == nullptr)
        SetRetCode(TextError);
    else if (pCurrent->pNext == nullptr)
        SetRetCode(TextNoNext);
    else
        pCurrent->pNext = pCurrent->pNext->pNext;
}

        /*-------------------------------------------*/

// итератор
int TText::Reset(void) { // установить на первую запись
    while (!St.empty())
        St.pop();
    pCurrent = pFirst;
    if (pCurrent != nullptr) {
        St.push(pCurrent);
        if (pCurrent->pNext != nullptr)
            St.push(pCurrent->pNext);
        if (pCurrent->pDown != nullptr)
            St.push(pCurrent->pDown);
    }
    return IsTextEnded();
}

int TText::IsTextEnded(void) const { // текст завершен?
    return !St.size();
}

int TText::GoNext(void) { // Переход к следующей записи
    if (!IsTextEnded()) {
        pCurrent = St.top(); St.pop();
        if (pCurrent != pFirst) {
            if (pCurrent->pNext != nullptr)
                St.push(pCurrent->pNext);
            if (pCurrent->pDown !=nullptr)
                St.push(pCurrent->pDown);
        }
    }
    return IsTextEnded();
}

        /*-------------------------------------------*/

// копирование текста
PTTextLink TText::GetFirstAtom(PTTextLink pl) { // поиск первого атома
    PTTextLink tmp = pl;
    while (!tmp->IsAtom()) {
        St.push(tmp);
        tmp = tmp->GetDown();
    }
    return tmp;
}

PTText TText::GetCopy() { // копирование текста
    PTTextLink pl1, pl2, pl = pFirst, cpl = nullptr;

    if (pFirst != nullptr) {
        while (!St.empty())
            St.pop(); // очистка стека
        while (true) {
            if (pl != nullptr) { // переход к первому атому
                pl = GetFirstAtom(pl);
                St.push(pl);
                pl = pl->GetDown();
            }
            else if (St.empty()) break;
            else {
                pl1 = St.top(); St.pop();
                if (strstr(pl1->Str, "Copy") == nullptr) { // первый этап создания копии
                    // создание копии - pDown на уже скопированный подуровень
                    pl2 = new TTextLink("Copy", pl1, cpl); // pNext на оригинал
                    St.push(pl2);
                    pl = pl1->GetNext();
                    cpl = nullptr;
                }
                else { // второй этап создания копии
                    pl2 = pl1->GetNext();
                    strcpy_s(pl1->Str, pl2->Str);
                    pl1->pNext = cpl;
                    cpl = pl1;
                }
            }
        }
    }
    return new TText(cpl);
}

        /*-------------------------------------------*/

    // печать текста
void TText::Print() {
    TextLevel = 0;
    PrintText(pFirst);
    Reset();
}

void TText::PrintText(PTTextLink ptl) {
    if (ptl != nullptr) {
        for (int i = 0; i < TextLevel; i++)
            cout << " ";
        cout << ptl->Str << endl;
        TextLevel++; PrintText(ptl->GetDown());
        TextLevel--; PrintText(ptl->GetNext());
    }
}

void TText::PrintTextFile(PTTextLink ptl, ofstream& txtFile)
{
    if (ptl != nullptr) {
        for (int i = 0; i < TextLevel; i++)
            txtFile << " ";
        txtFile << ptl->Str << endl;
        TextLevel++; PrintTextFile(ptl->GetDown(), txtFile);
        TextLevel--; PrintTextFile(ptl->GetNext(), txtFile);
    }
}

void TText::Write(string pFileName)
{
    TextLevel = 0;
    ofstream TextFile(pFileName);
    PrintTextFile(pFirst, TextFile);
    Reset();
}

        /*-------------------------------------------*/

// чтение текста
void TText::Read(string pFileName) {
    ifstream txtFile(pFileName);
    TextLevel = 0;
    if (&txtFile != nullptr)
        pFirst = ReadText(txtFile);
    Reset();
}

PTTextLink TText::ReadText(ifstream& TxtFile)
{
    string buf;
    PTTextLink ptl = new TTextLink();
    PTTextLink tmp = ptl;
    while (!TxtFile.eof())
    {
        getline(TxtFile, buf);
        if (buf.front() == '}')
            break;
        else if (buf.front() == '{')
            ptl->pDown = ReadText(TxtFile);
        else
        {
            ptl->pNext = new TTextLink(buf.c_str());
            ptl = ptl->pNext;
        }
    }
    ptl = tmp;
    if (tmp->pDown == nullptr)
    {
        tmp = tmp->pNext;
        delete ptl;
    }
    return tmp;
}

```

__Тесты для класса `TTextLink`:__

```c++

#include "gtest/gtest.h"

#include <iostream>
#include <src/TText.cpp>

TEST(TText, Can_Create_Empty_Text) {
    ASSERT_NO_THROW(TText _text);
}

TEST(TText, Can_Create_Not_EmptyText) {
    const string str = "qqq";

    TTextLink::InitMemSystem(1);
    TTextLink Link = TTextLink(str);

    ASSERT_NO_THROW(TText _text(&Link));
}

TEST(TText, RetCode_Of_New_Text_Is_OK) {
    const string str = "qqq";

    TTextLink::InitMemSystem(1);
    TTextLink Link = TTextLink(str);
    TText _text(&Link);

    ASSERT_TRUE(_text.GetRetCode() == TextOK);
}

TEST(TText, Can_Set_Current_Line) {
    const string str1 = "qqq", str2 = "abc";
    TTextLink::InitMemSystem(10);
    TTextLink Link = TTextLink(str1);
    TText _text(&Link);
    _text.GoFirstLink();

    ASSERT_NO_THROW(_text.SetLine(str2));
}

TEST(TText, Can_Get_Current_Line)
{
    const string str1 = "qqq", str2 = "abc";
    TTextLink::InitMemSystem(10);
    TTextLink Link = TTextLink(str1);
    TText _text(&Link);
    _text.GoFirstLink();

    _text.SetLine(str2);
    bool equality_of_strings = (_text.GetLine() == str2) 
                            && (_text.GetLine() != str1);

    ASSERT_TRUE(equality_of_strings);
}

TEST(TText, Can_Go_Down_Link) {
    const string str1 = "Section 1", str2 = "Section 1.1";
    TTextLink::InitMemSystem(10);
    TTextLink Link = TTextLink(str1);
    TText _text(&Link);
    _text.GoFirstLink();

    _text.InsDownLine(str2);
    _text.GoDownLink();

    EXPECT_EQ(str2, _text.GetLine());
}

TEST(TText, Can_Go_Next_Link) {
    const string str1 = "Section 1", str2 = "Section 2";
    TTextLink::InitMemSystem(10);
    TTextLink Link = TTextLink(str1);
    TText _text(&Link);
    _text.GoFirstLink();

    _text.InsNextLine(str2);
    _text.GoNextLink();

    EXPECT_EQ(str2, _text.GetLine());
}

TEST(TText, Can_Go_First_Link) {
    string str = "Section 1";
    TTextLink::InitMemSystem(10);
    TTextLink Link = TTextLink(str);
    TText _text(&Link);
    _text.GoFirstLink();

    for (int i = 0; i < 3; i++) {
        str += "1";
        _text.InsNextLine(str);
        _text.GoNextLink();
    }

    bool isFirstLink = (_text.GetLine() == "Section 1111");
    _text.GoFirstLink();
    isFirstLink = isFirstLink && (_text.GetLine() == "Section 1");
    ASSERT_TRUE(isFirstLink);
}

TEST(TText, Can_Go_Previous_Link) {
    string str = "Section 1";
    TTextLink::InitMemSystem(10);
    TTextLink Link = TTextLink(str);
    TText _text(&Link);
    _text.GoFirstLink();

    for (int i = 0; i < 3; i++) {
        str += "1";
        _text.InsNextLine(str);
        _text.GoNextLink();
    }

    _text.GoPrevLink();
    EXPECT_EQ("Section 111", _text.GetLine());
}

TEST(TText, Set_Error_Code_When_Go_Nonexistent_Link) {
    const string str1 = "Section 1";
    TTextLink::InitMemSystem(10);
    TTextLink Link = TTextLink(str1);
    TText _text(&Link); _text.GoFirstLink();
    bool error_codes;

    _text.GoDownLink();
    error_codes = _text.GetRetCode() == TextNoDown;
    _text.GoNextLink();
    error_codes = error_codes && (_text.GetRetCode() == TextNoNext);

    ASSERT_TRUE(error_codes);
}


TEST(TText, Can_Delete_Down_Line)
{
    // Arrange
    const string str1 = "Hello world!";
    const string str2 = "Hello!";
    const string str3 = "World welcomes you";
    TTextLink::InitMemSystem(4);
    TText T;
    T.GoFirstLink();
    T.InsDownLine(str1);
    T.GoDownLink();
    T.InsDownLine(str2);
    T.InsDownLine(str3);

    T.GoDownLink();
    T.DelDownLine();

    T.GoFirstLink();
    T.GoDownLink();
    EXPECT_EQ(T.GetLine(), str1);
    T.GoDownLink();
    EXPECT_EQ(T.GetLine(), str3);
}

TEST(TText, Can_Delete_Down_Section)
{
    const string sec1 = "section 1";
    const string sec2 = "section 1.1";
    const string sec3 = "section 1.2";
    const string str1 = "string 1";
    const string str2 = "string 2";
    TTextLink::InitMemSystem(6);
    TText T;
    T.GoFirstLink();
    T.InsDownLine(sec1);
    T.GoDownLink();
    T.InsDownLine(sec3);
    T.InsDownLine(sec2);
    T.GoDownLink();
    T.InsDownLine(str2);
    T.InsDownLine(str1);

    T.DelDownSection();

    T.GoFirstLink();
    T.GoDownLink();
    T.GoDownLink();
    EXPECT_EQ(T.GoDownLink(), TextNoDown);
}

TEST(TText, Can_Delete_Next_Line)
{
    const string sec1 = "section 1";
    const string str1 = "string 1";
    const string str2 = "string 3";
    const string str3 = "string 2";
    TTextLink::InitMemSystem(5);
    TText T;
    T.GoFirstLink();
    T.InsDownLine(sec1);
    T.GoDownLink();
    T.InsDownSection(str1);
    T.GoDownLink();
    T.InsNextLine(str2);
    T.InsNextLine(str3);

    T.DelNextLine();

    T.GoFirstLink();
    T.GoDownLink();
    T.GoDownLink();
    bool equality_of_strings = T.GetLine() == str1;
    T.GoNextLink();
    equality_of_strings = equality_of_strings && (T.GetLine() == str2);

    ASSERT_TRUE(equality_of_strings);
}

TEST(TText, Can_Delete_Next_Section)
{
    const string sec1 = "section 1";
    const string sec2 = "section 2";
    const string str1 = "string 1";
    const string str2 = "string 2";
    const string str3 = "string 3";
    TTextLink::InitMemSystem(7);
    TText T;

    T.GoFirstLink();
    T.InsDownLine(sec1);
    T.GoDownLink();
    T.InsNextLine(str2);
    T.InsNextLine(str1);
    T.InsNextSection(sec2);
    T.InsNextLine(str3);

    T.GoNextLink();
    T.DelNextSection();

    T.GoFirstLink();
    T.GoDownLink();
    T.GoNextLink();
    EXPECT_EQ(T.GetLine(), str3);
}

TEST(TText, Can_Use_Iterator) {
    string str = "Section 1";
    TTextLink::InitMemSystem(10);
    TTextLink Link = TTextLink(str);
    TText _text(&Link); _text.GoFirstLink();
    for (int i = 0; i < 3; i++) {
        _text.InsNextLine(str);
        _text.GoNextLink();
        str += "1";
    }

    bool correctness = true;
    str = "Section 1";
    for (_text.Reset(); _text.IsTextEnded(); _text.GoNext()) {
        correctness = correctness && (str == _text.GetLine());
        correctness = correctness && !_text.IsTextEnded();
        str += "1";
    }

    ASSERT_TRUE(correctness);
}

```

__Подтверждение успешного прохождения тестов:__

![Tests.jpg](http://i.imgur.com/zcFNxiX.jpg "Tests.jpg")

__Демонстрационная программа:__

```c++
// TextTestKit.cpp - пример использования класса TText

#include "include/TText.h"

int main(int argc, char* argv[]) {
    setlocale(LC_ALL, "Russian");
    string fileName;

    if (argc < 2) {
        cout << "Введите имя файла с текстом >> ";
        cin >> fileName;
        cout << endl;
    }
    else
        fileName = argv[1];

    PTText pText;
    TTextLink::InitMemSystem(50);
    TText text;

    text.Read(fileName);
    cout << "Текст, прочитанный из файла " << fileName << ":\n";
    cout << endl;
    text.Print();
    cout << endl;

    cout << "Модифицированный текст :\n";
    cout << endl;
    pText = text.GetCopy();
    pText->GoFirstLink();
    pText->SetLine("АИСД. 2 курс");
    pText->InsDownLine("Раздел 1");
    pText->GoDownLink();
    pText->InsDownLine("1.1 Введение");
    pText->GoDownLink();
    pText->InsNextLine("1.2 Битовый массив");
    pText->GoNextLink();
    pText->InsDownLine("1. Определение");
    pText->GoDownLink();
    pText->InsNextLine("2. Структура");
    pText->GoFirstLink();
    pText->GoDownLink();
    pText->GoNextLink();
    pText->GoDownLink();
    pText->GoDownLink();
    pText->InsNextLine("2. Применение");
    pText->GoNextLink();
    pText->DelNextLine();
    pText->GoPrevLink();
    pText->GoPrevLink();
    pText->DelNextSection();
    pText->Print();
    
    fileName = "output.txt";
    pText->Write(fileName);
    cout << "\nМодифицированный текст сохранен в файл " << fileName << endl;
    return 0;
}
```

__Результат работы:__

![TestKit.jpg](http://i.imgur.com/4pT1NqV.jpg "TestKit.jpg")

![TestKit_output.jpg](http://i.imgur.com/bC4dysc.jpg "TestKit_output.jpg")

## Используемые инструменты

- Система контроля версий [Git][git].
- Фреймворк для написания автоматических тестов [Google Test][gtest].
- Среда разработки Microsoft Visual Studio 2013, Visual Studio 2015.

## Вывод

Выполнение данной работы помогло получить начальные навыки командной разработки. Стоит отметить, что неправильное распределение обязанностей может существенно замедлить разработку. Поэтому важно распределять обязанности, учитывая знания и умения каждого члена команды.  

Результатом работы стал класс, который в перспективе может быть использован для создания текстового редактора. Класс может использоваться и без "обертки" в виде текстового редактора, однако для совершения довольно простых действий могут потребоваться достаточно громоздкие конструкции из однотипных обращений к методам класса.

<!-- LINKS -->

[git]:         https://git-scm.com/book/ru/v2
[gtest]:       https://github.com/google/googletest
[sieve]:       http://habrahabr.ru/post/91112
[git-guide]:   https://bitbucket.org/ashtan/mp2-lab1-set/src/ff6d76c3dcc2a531cefdc17aad5484c9bb8b47c5/docs/part1-git.md?at=master&fileviewer=file-view-default
[gtest-guide]: https://bitbucket.org/ashtan/mp2-lab1-set/src/ff6d76c3dcc2a531cefdc17aad5484c9bb8b47c5/docs/part2-google-test.md?at=master&fileviewer=file-view-default
[upstream]:    https://bitbucket.org/ashtan/mp2-lab1-set 