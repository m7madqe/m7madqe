#include <iostream>
#include <fstream>
#include <sstream>
#include <string>
#include <map>
#include <limits>
#include <cstdlib>

using namespace std;

// تعريف ألوان ANSI
const string BLUE = "\033[1;34m";
const string LIGHT_ORANGE_BG = "\033[48;5;215m";
const string RESET = "\033[0m";
const string CLEAR_SCREEN = "\033[2J\033[H";  // مسح الشاشة وإعادة المؤشر للبداية

// تعريف هيكل بيانات الشخص
struct Person {
    string name;
    int age;
    string profession;
    string address;
    string phone;
};

// دالة لمسح الشاشة وعرض القائمة في أعلى الصفحة
void displayMenu() {
    cout << CLEAR_SCREEN;  // تنظيف الشاشة
    cout << "===================================\n";
    cout << "      " << BLUE << "نظام إدارة المعلومات" << RESET << "\n";
    cout << "===================================\n";
    cout << "1. إضافة سجل جديد\n";
    cout << "2. عرض معلومات سجل\n";
    cout << "3. تحديث سجل\n";
    cout << "4. حذف سجل\n";
    cout << "5. عرض جميع السجلات\n";
    cout << "6. حفظ البيانات والخروج\n";
    cout << "-----------------------------------\n";
    cout << "اختر خيارًا: ";
}

// إدارة السجلات
class RecordManager {
private:
    map<string, Person> records;
    const string filename = "records.txt";

public:
    void loadRecords() {
        ifstream inFile(filename);
        if (!inFile) return;
        string line;
        while (getline(inFile, line)) {
            if (line.empty()) continue;
            stringstream ss(line);
            Person p;
            getline(ss, p.name, '|');
            ss >> p.age;
            ss.ignore();
            getline(ss, p.profession, '|');
            getline(ss, p.address, '|');
            getline(ss, p.phone);
            records[p.name] = p;
        }
        inFile.close();
    }

    void saveRecords() {
        ofstream outFile(filename);
        for (const auto &pair : records) {
            outFile << pair.second.name << "|" << pair.second.age << "|" 
                    << pair.second.profession << "|" << pair.second.address 
                    << "|" << pair.second.phone << "\n";
        }
        outFile.close();
    }

    void addRecord() {
        Person p;
        cout << "أدخل الاسم: ";
        cin >> p.name;
        cout << "أدخل العمر: ";
        cin >> p.age;
        cin.ignore();
        cout << "أدخل المهنة: ";
        getline(cin, p.profession);
        cout << "أدخل العنوان: ";
        getline(cin, p.address);
        cout << "أدخل رقم الهاتف: ";
        getline(cin, p.phone);
        records[p.name] = p;
        cout << "تمت إضافة السجل بنجاح!\n";
    }

    void displayRecord() {
        cout << "أدخل الاسم للبحث: ";
        string name;
        cin >> name;
        if (records.find(name) != records.end()) {
            Person p = records[name];
            cout << "\n=== تفاصيل السجل ===\n";
            cout << "الاسم: " << LIGHT_ORANGE_BG << BLUE << p.name << RESET << "\n";
            cout << "العمر: " << p.age << "\n";
            cout << "المهنة: " << p.profession << "\n";
            cout << "العنوان: " << p.address << "\n";
            cout << "رقم الهاتف: " << p.phone << "\n";
        } else {
            cout << "السجل غير موجود!\n";
        }
    }

    void updateRecord() {
        cout << "أدخل الاسم لتحديث السجل: ";
        string name;
        cin >> name;
        if (records.find(name) != records.end()) {
            cout << "أدخل العمر الجديد: ";
            cin >> records[name].age;
            cin.ignore();
            cout << "أدخل المهنة الجديدة: ";
            getline(cin, records[name].profession);
            cout << "أدخل العنوان الجديد: ";
            getline(cin, records[name].address);
            cout << "أدخل رقم الهاتف الجديد: ";
            getline(cin, records[name].phone);
            cout << "تم تحديث السجل!\n";
        } else {
            cout << "السجل غير موجود!\n";
        }
    }

    void deleteRecord() {
        cout << "أدخل الاسم لحذف السجل: ";
        string name;
        cin >> name;
        if (records.erase(name)) {
            cout << "تم حذف السجل بنجاح!\n";
        } else {
            cout << "السجل غير موجود!\n";
        }
    }

    void displayAllRecords() {
        if (records.empty()) {
            cout << "لا توجد سجلات.\n";
            return;
        }
        cout << "\n=== جميع السجلات ===\n";
        for (const auto &pair : records) {
            cout << "------------------------\n";
            cout << "الاسم: " << LIGHT_ORANGE_BG << BLUE << pair.second.name << RESET << "\n";
            cout << "العمر: " << pair.second.age << "\n";
            cout << "المهنة: " << pair.second.profession << "\n";
            cout << "العنوان: " << pair.second.address << "\n";
            cout << "رقم الهاتف: " << pair.second.phone << "\n";
        }
        cout << "------------------------\n";
    }
};

int main() {
    RecordManager manager;
    manager.loadRecords();
    int choice;

    do {
        displayMenu();
        while (!(cin >> choice)) {
            cout << "الرجاء إدخال رقم صحيح: ";
            cin.clear();
            cin.ignore(numeric_limits<streamsize>::max(), '\n');
        }
        cout << CLEAR_SCREEN;  // مسح الشاشة عند كل اختيار لتحديث العرض
        switch(choice) {
            case 1:
                manager.addRecord();
                break;
            case 2:
                manager.displayRecord();
                break;
            case 3:
                manager.updateRecord();
                break;
            case 4:
                manager.deleteRecord();
                break;
            case 5:
                manager.displayAllRecords();
                break;
            case 6:
                manager.saveRecords();
                cout << "تم حفظ البيانات، وداعًا!\n";
                break;
            default:
                cout << "خيار غير صالح. الرجاء اختيار رقم من 1 إلى 6.\n";
        }
        cout << "\nاضغط على Enter للعودة إلى القائمة...";
        cin.ignore();
        cin.get();  // انتظار إدخال المستخدم قبل إعادة عرض القائمة

    } while(choice != 6);

    return 0;
}
