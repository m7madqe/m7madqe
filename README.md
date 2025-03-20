#include <iostream>
#include <fstream>
#include <sstream>
#include <string>
#include <map>
#include <limits>
#include <cstdlib>   // لاستخدام system()

using namespace std;

// تعريف أكواد الألوان باستخدام ANSI escape codes
const string BLUE = "\033[1;34m";
const string LIGHT_ORANGE_BG = "\033[48;5;215m";
const string RESET = "\033[0m";

// تعريف هيكل Person لتمثيل بيانات الشخص مع معلومات إضافية
struct Person {
    string name;
    int age;
    string profession;
    string address;
    string phone;
};

// دالة لترميز المسافات في النص ليناسب عنوان URL
string urlEncode(const string &value) {
    string encoded;
    for (char c : value) {
        if (c == ' ') {
            encoded += "%20";
        } else {
            encoded.push_back(c);
        }
    }
    return encoded;
}

// دالة لإرسال رسالة واتساب عبر المتصفح باستخدام رابط واتساب للرقم +962790542645
void sendWhatsAppMessage(const Person &p) {
    // تكوين نص الرسالة
    string message = "اسم: " + p.name +
                     "، العمر: " + to_string(p.age) +
                     "، المهنة: " + p.profession +
                     "، العنوان: " + p.address +
                     "، رقم الهاتف: " + p.phone;
    // ترميز النص ليناسب الرابط
    string encodedMessage = urlEncode(message);
    // تكوين رابط واتساب
    string url = "https://wa.me/+962790542645?text=" + encodedMessage;
    
    // فتح الرابط باستخدام الأمر النظامي
    // في Windows استخدم "start"، وإذا كنت على Linux استبدله بـ "xdg-open"
    string command = "start " + url;
    system(command.c_str());
}

class RecordManager {
private:
    map<string, Person> records;
    const string filename = "records.txt";

    // تحويل سجل إلى سلسلة نصية للتخزين في الملف
    string serialize(const Person &p) {
        return p.name + "|" + to_string(p.age) + "|" + p.profession + "|" + p.address + "|" + p.phone;
    }
    
    // تحويل السلسلة النصية إلى سجل Person
    Person deserialize(const string &line) {
        Person p;
        stringstream ss(line);
        getline(ss, p.name, '|');
        string ageStr;
        getline(ss, ageStr, '|');
        p.age = stoi(ageStr);
        getline(ss, p.profession, '|');
        getline(ss, p.address, '|');
        getline(ss, p.phone);
        return p;
    }

public:
    // تحميل السجلات من الملف عند بدء التشغيل
    void loadRecords() {
        ifstream inFile(filename);
        if (!inFile) {
            cout << "لا يوجد ملف سجلات موجود، سيتم إنشاء ملف جديد عند الحفظ.\n";
            return;
        }
        string line;
        while (getline(inFile, line)) {
            if (line.empty()) continue;
            Person p = deserialize(line);
            records[p.name] = p;
        }
        inFile.close();
    }

    // حفظ السجلات في الملف
    void saveRecords() {
        ofstream outFile(filename);
        for (const auto &pair : records) {
            outFile << serialize(pair.second) << "\n";
        }
        outFile.close();
    }

    // إضافة سجل جديد وإرسال بياناته عبر واتساب
    void addRecord() {
        Person p;
        cout << "أدخل الاسم: ";
        cin >> p.name;
        
        cout << "أدخل العمر: ";
        while (!(cin >> p.age)) {
            cout << "الرجاء إدخال رقم صحيح للعمر: ";
            cin.clear();
            cin.ignore(numeric_limits<streamsize>::max(), '\n');
        }
        cin.ignore(numeric_limits<streamsize>::max(), '\n');
        
        cout << "أدخل المهنة: ";
        getline(cin, p.profession);
        
        cout << "أدخل العنوان: ";
        getline(cin, p.address);
        
        cout << "أدخل رقم الهاتف: ";
        getline(cin, p.phone);

        records[p.name] = p;
        cout << "تمت إضافة السجل بنجاح.\n";

        // إرسال البيانات عبر واتساب
        sendWhatsAppMessage(p);
    }

    // عرض سجل محدد بحسب الاسم مع تلوين الاسم بنص أزرق وخلفية برتقالية فاتحة
    void displayRecord() {
        cout << "أدخل الاسم للبحث: ";
        string name;
        cin >> name;
        auto it = records.find(name);
        if (it != records.end()) {
            cout << "\nتفاصيل السجل:\n";
            cout << "الاسم: " << LIGHT_ORANGE_BG << BLUE << it->second.name << RESET << "\n";
            cout << "العمر: " << it->second.age << "\n";
            cout << "المهنة: " << it->second.profession << "\n";
            cout << "العنوان: " << it->second.address << "\n";
            cout << "رقم الهاتف: " << it->second.phone << "\n";
        } else {
            cout << "السجل غير موجود.\n";
        }
    }

    // تحديث سجل موجود وإرسال البيانات المحدثة عبر واتساب
    void updateRecord() {
        cout << "أدخل الاسم لتحديث السجل: ";
        string name;
        cin >> name;
        auto it = records.find(name);
        if (it != records.end()) {
            cout << "أدخل العمر الجديد: ";
            while (!(cin >> it->second.age)) {
                cout << "الرجاء إدخال رقم صحيح للعمر: ";
                cin.clear();
                cin.ignore(numeric_limits<streamsize>::max(), '\n');
            }
            cin.ignore(numeric_limits<streamsize>::max(), '\n');
            
            cout << "أدخل المهنة الجديدة: ";
            getline(cin, it->second.profession);
            
            cout << "أدخل العنوان الجديد: ";
            getline(cin, it->second.address);
            
            cout << "أدخل رقم الهاتف الجديد: ";
            getline(cin, it->second.phone);
            
            cout << "تم تحديث السجل.\n";
            // إرسال البيانات المحدثة عبر واتساب
            sendWhatsAppMessage(it->second);
        } else {
            cout << "السجل غير موجود.\n";
        }
    }

    // حذف سجل
    void deleteRecord() {
        cout << "أدخل الاسم لحذف السجل: ";
        string name;
        cin >> name;
        if (records.erase(name))
            cout << "تم حذف السجل.\n";
        else
            cout << "السجل غير موجود.\n";
    }

    // عرض جميع السجلات مع تلوين الأسماء بنص أزرق وخلفية برتقالية فاتحة
    void displayAllRecords() {
        if (records.empty()) {
            cout << "لا توجد سجلات.\n";
            return;
        }
        cout << "\nجميع السجلات:\n";
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

void displayMenu() {
    cout << "\n====== نظام إدارة المعلومات ======\n";
    cout << "1. إضافة سجل\n";
    cout << "2. عرض معلومات سجل\n";
    cout << "3. تحديث سجل\n";
    cout << "4. حذف سجل\n";
    cout << "5. عرض جميع السجلات\n";
    cout << "6. حفظ البيانات والخروج\n";
    cout << "اختر خياراً: ";
}

int main() {
    RecordManager manager;
    manager.loadRecords();
    int choice;

    do {
        displayMenu();
        while (!(cin >> choice)) {
            cout << "الرجاء إدخال رقم صحيح للخيار: ";
            cin.clear();
            cin.ignore(numeric_limits<streamsize>::max(), '\n');
        }
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
    } while(choice != 6);
    
    return 0;
} 
