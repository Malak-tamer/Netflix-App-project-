# Night Flix-App-
#include <iostream>
#include <fstream>
#include <sstream>
#include <string>
#include <queue>
#include <cc
using namespace std;

// ======== Helpers ========
string normalize(string s) {
    for (auto& c : s) c = tolower(c);
    return s;
}

// ======== USER MANAGER WITH FILE ========
class usernode {
public:
    string username;
    string password;
    int age;
    usernode* next;
    usernode(string u, string p, int a = 0) {
        username = u;
        password = p;
        age = a;
        next = nullptr;
    }
};

class usermanger {
private:
    usernode* head;
    string filename = "usersdata.txt";
public:
    usermanger() {
        head = nullptr;
        loaduser();
    }
    ~usermanger() {
        saveusers();
        while (head) {
            usernode* temp = head;
            head = head->next;
            delete temp;
        }
    }

    void loaduser() {
        ifstream file(filename);
        if (!file.is_open()) return;
        string line;
        while (getline(file, line)) {
            if (line.empty()) continue;
            stringstream ss(line);
            string u, p, a_str;
            if (!getline(ss, u, ',')) continue;
            if (!getline(ss, p, ',')) continue;
            if (!getline(ss, a_str)) continue;
            int a = stoi(a_str);
            usernode* newuser = new usernode(u, p, a);
            newuser->next = head;
            head = newuser;
        }
        file.close();
    }

    void saveusers() {
        ofstream file(filename);
        usernode* current = head;
        while (current) {
            file << current->username << "->" << current->password << "->" << current->age << endl;
            current = current->next;
        }
        file.close();
    }

    void signin(string username, string password, int age) {
        usernode* c = head;
        while (c != nullptr) {
            if (c->username == username) {
                cout << "Username already exists !!" << endl;
                return;
            }
            c = c->next;
        }
        usernode* newuser = new usernode(username, password, age);
        newuser->next = head;
        head = newuser;
        saveusers();
        cout << "User signed in successfully (:" << endl;
    }

    usernode* login(string username, string password) {
        usernode* cur = head;
        while (cur != nullptr) {
            if (cur->username == username && cur->password == password) {
                cout << "Login successful ! Welcome " << username << "(: " << endl;
                return cur;
            }
            cur = cur->next;
        }
        cout << "Incorrect username or password . Try again ): " << endl;
        return nullptr;
    }

    void displayusers() {
        usernode* cur = head;
        cout << "Users : " << endl;
        while (cur) {
            cout << "- Username : " << cur->username << " (Age: " << cur->age << ")" << endl;
            cur = cur->next;
        }
    }
};

// ======== MOVIE BST ======== (MALAK YASSER )
class Movie {
public:
    string name;
    string description;
    int duration;
    int ageRating;
    string language;
    Movie* left;
    Movie* right;

    Movie(string n, string d, int dur, int age, string lang) {
        name = n;
        description = d;
        duration = dur;
        ageRating = age;
        language = lang;
        left = right = nullptr;
    }
};

class RecentQueue;

class MovieBST {
public:
    Movie* root;

    MovieBST() { root = nullptr; }

    Movie* insertRec(Movie* node, Movie* m) {
        if (!node) return m;
        if (m->name < node->name)
            node->left = insertRec(node->left, m);
        else
            node->right = insertRec(node->right, m);
        return node;
    }

    void addMovie(const string& n, const string& d, int dur, int age, const string& lang, RecentQueue* recent = nullptr);

    Movie* searchByName(Movie* node, const string& nameN) {
        if (!node) return nullptr;
        if (normalize(node->name) == nameN) return node;
        Movie* res = searchByName(node->left, nameN);
        if (res != nullptr) return res;
        return searchByName(node->right, nameN);
    }

    Movie* searchByName(const string& name) {
        return searchByName(root, normalize(name));
    }

    void displayAandL(Movie* node, int userAge, const string& langN) {
        if (!node) return;
        displayAandL(node->left, userAge, langN);
        if (node->ageRating <= userAge && normalize(node->language) == langN) {
            cout << "Name: " << node->name << endl;
            cout << "--------------" << endl;
            cout << "Duration: " << node->duration << " minutes" << endl;
            cout << "--------------" << endl;
            cout << "Age Rating: " << node->ageRating << endl;
            cout << "--------------" << endl;
            cout << "Description: " << node->description << endl;
            cout << "--------------" << endl;
            cout << "Language: " << node->language << endl << endl;
            cout << "----------------------------------" << endl;
        }
        displayAandL(node->right, userAge, langN);
    }

    void displayByAgeAndLang(int userAge, string lang) {
        displayAandL(root, userAge, normalize(lang));
    }
};

// ======== RECENTLY ADDED MOVIES ======== AHMED ELHOSSARY 
class RecentQueue {
private:
    queue<Movie*> q;
    int maxSize;
public:
    RecentQueue(int maxR = 5) { maxSize = maxR; }

    void add(Movie* m) {
        q.push(m);
        if (q.size() > maxSize) q.pop();
    }

    void display() {
        if (q.empty()) { cout << "No recent movies." << endl; return; }
        cout << "Recently Added Movies:" << endl;
        queue<Movie*> temp = q;
        while (!temp.empty()) {
            cout << temp.front()->name << endl;
            temp.pop();
        }
        cout << endl;
    }
};
// ======== IMPLEMENT addMovie with RecentQueue ======== 
void MovieBST::addMovie(const string& n, const string& d, int dur, int age, const string& lang, RecentQueue* recent) {
    Movie* newMovie = new Movie(n, d, dur, age, lang);
    root = insertRec(root, newMovie);
    if (recent) recent->add(newMovie);
}

// ======== WATCHLIST (Doubly Linked List) ======== TAREQ OSAMA
class WatchNode {
public:
    string movieName;
    WatchNode* next;
    WatchNode* prev;

    WatchNode(string n) {
        movieName = n;
        next = prev = nullptr;
    }
};
class Watchlist {
public:
    WatchNode* head;
    WatchNode* tail;

    Watchlist() { head = tail = nullptr; }

    void add(string movieName) {
        WatchNode* newNode = new WatchNode(movieName);
        if (!head) head = tail = newNode;
        else {
            tail->next = newNode;
            newNode->prev = tail;
            tail = newNode;
        }
        cout << "Added to watchlist!" << endl;
    }

    void display() {
        if (!head) {
            cout << "Watchlist is Empty!" << endl;
            return;
        }
        WatchNode* temp = head;
        cout << "Your Watchlist:" << endl;
        while (temp) {
            cout << temp->movieName;
            if (temp->next) cout << " -> ";
            temp = temp->next;
        }
        cout << " -> END" << endl << endl;
    }
};


// ======== SAMPLE DATA ======== MALAK TAMER 
void initSampleData(MovieBST& movies, RecentQueue& recent) {
    //+3 age 
    movies.addMovie("Coco", "A boy enters the spirit world.", 105, 3, "english", &recent);
    movies.addMovie("Frozen", "Anna saves Elsa.", 95, 3, "english", &recent);
    movies.addMovie("Bee movie ", "a bee seeking justice in the human world ", 120, 3, "english", &recent);
    movies.addMovie("Ladybug", "heart warming moments for children ", 124, 3, "english", &recent);

    //+13 
    movies.addMovie("Back in action ", "Action and comedy for teens  ", 145, 13, "english", &recent);
    movies.addMovie("Elharifa ", "Football teamwork story ", 117, 13, "arabic", &recent);//arabic 
    movies.addMovie("Harry Botter ", "Magical story ", 147, 13, "english", &recent);
    movies.addMovie("The Dark Knight", "Batman vs Joker.", 120, 13, "english", &recent);
    movies.addMovie("Siko siko ", "Two men sstruggle to find their money ", 119, 13, "arabic", &recent);//arabic 
    movies.addMovie("Face to face ", "Emotional humman story ", 134, 13, "arabic", &recent);//arabic 
    //+18

    movies.addMovie("IT ", " Classic story clown Horror ", 120, 18, "english", &recent);
    movies.addMovie("The Nun", "A scary demonic movie.", 110, 18, "english", &recent);
    movies.addMovie("Escape room ", " Suspense story ", 127, 18, "english", &recent);
    movies.addMovie("Asrar el qahra  ", "Horror story in old cairo ", 114, 18, "arabic", &recent);//Arabic 
    movies.addMovie("Elfel el azraa ", "physiological horror", 123, 18, "arabic", &recent);//arabic 




}

// ======== MENUS ========
bool guestMenu(usermanger& users, usernode*& currentUser) {
    cout << "--------------------------------------" << endl;
    cout << "------- Welcome to Night Flix ---------" << endl;
    cout << "--------------------------------------" << endl;
    cout << "1) Login " << endl;
    cout << "2) Create Account" << endl;
    cout << "3) Exit " << endl;
    cout << "Choice: ";

    int ch;
    if (!(cin >> ch)) { cin.clear(); string t; getline(cin, t); return true; }
    cin.ignore();
    if (ch == 3) return false;

    if (ch == 1) {
        string u, p;
        cout << "Username: ";
        getline(cin, u);
        cout << "Password: ";
        getline(cin, p);
        currentUser = users.login(u, p);
    }
    else if (ch == 2) {
        string u, p; int age;
        cout << "Username: "; getline(cin, u);
        cout << "Password: "; getline(cin, p);
        cout << "Age: "; cin >> age; cin.ignore();
        users.signin(u, p, age);
    }
    return true;
}
// NAIRA AHMED 
void userMenu(usernode*& currentUser, MovieBST& movies, Watchlist& watch, RecentQueue& recent) {
    cout << "Hello " << currentUser->username << " (Age: " << currentUser->age << ")" << endl;
    cout << "1) Change Age" << endl;
    cout << "2) Browse Movies" << endl;
    cout << "3) Recent Movies" << endl;
    cout << "4) Watchlist" << endl;
    cout << "5) Logout" << endl;
    cout << "-------------" << endl;
    cout << "Choice: ";

    int ch;
    if (!(cin >> ch)) { cin.clear(); string t; getline(cin, t); return; }
    cin.ignore();

    if (ch == 1) {
        cout << "New age: "; cin >> currentUser->age; cin.ignore();
    }
    else if (ch == 2) {
        string lang;
        cout << "Language: "; getline(cin, lang);
        movies.displayByAgeAndLang(currentUser->age, lang);

        string m;
        cout << "Add a movie to your watchlist (or empty to skip): ";
        getline(cin, m);
        if (!m.empty()) {
            Movie* f = movies.searchByName(m);
            if (f) watch.add(f->name);
            else cout << "Movie not found" << endl;
        }
    }
    else if (ch == 3) recent.display();
    else if (ch == 4) watch.display();
    else if (ch == 5) currentUser = nullptr;
}

void mainMenu(usermanger& users, MovieBST& movies, Watchlist& watch, RecentQueue& recent) {
    usernode* currentUser = nullptr;
    bool running = true;
    while (running) {
        if (!currentUser) running = guestMenu(users, currentUser);
        else userMenu(currentUser, movies, watch, recent);
    }
}

// ======== MAIN ========
int main() {
    usermanger users;
    MovieBST movies;
    Watchlist watch;
    RecentQueue recent;

    initSampleData(movies, recent);
    mainMenu(users, movies, watch, recent);

    return 0;
}
