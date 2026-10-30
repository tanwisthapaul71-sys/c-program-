#include <iostream>
#include <fstream>
#include <iomanip>
#include <vector>
#include <memory>
#include <string>
#include <stdexcept>
#include <limits>
#include <sstream>
using namespace std;

// Abstract class
class Advisory {
public:
    virtual void giveAdvice() const = 0;
    virtual ~Advisory() {}
};

// Base class
class Crop : public Advisory {
protected:
    string name, season;
    double area, yieldPerAcre;

    static int cropCount;

public:
    // Constructor overloading
    Crop(string n, double a, string s, double y) {
        if (a <= 0 || y <= 0)
            throw invalid_argument("Area and yield must be positive.");

        name = n;
        area = a;
        season = s;
        yieldPerAcre = y;
        cropCount++;
    }

    Crop(string n, double a, double y)
        : Crop(n, a, "General", y) {}

    // Virtual destructor
    virtual ~Crop() {
        cropCount--;
    }

    // Virtual function for runtime polymorphism
    virtual double calculateYield() const {
        return area * yieldPerAcre;
    }

    virtual void display() const {
        cout << "\nCrop Name: " << name;
        cout << "\nArea: " << area << " acres";
        cout << "\nSeason: " << season;
        cout << "\nYield per Acre: " << yieldPerAcre;
        cout << "\nTotal Yield: " << calculateYield() << " units\n";
    }

    // Operator overloading
    bool operator>(const Crop& c) const {
        return calculateYield() > c.calculateYield();
    }

    // Static member function
    static int getCropCount() {
        return cropCount;
    }

    // File saving function
    virtual void save(ofstream& file) const = 0;
};

int Crop::cropCount = 0;

// Derived class 1
class CerealCrop : public Crop {
    string fertilizerType;

public:
    CerealCrop(string n, double a, string s,
               double y, string f)
        : Crop(n, a, s, y) {
        fertilizerType = f;
    }

    // Overloaded constructor
    CerealCrop(string n, double a, double y)
        : Crop(n, a, y) {
        fertilizerType = "Organic";
    }

    void display() const override {
        cout << "\n--- Cereal Crop Details ---";
        Crop::display();
        cout << "Fertilizer Type: " << fertilizerType << endl;
    }

    void giveAdvice() const override {
        cout << "Advice: Use balanced fertilizers, "
             << "proper irrigation and quality seeds.\n";
    }

    void save(ofstream& file) const override {
        file << "CEREAL " << quoted(name) << " "
             << area << " " << quoted(season) << " "
             << yieldPerAcre << " "
             << quoted(fertilizerType) << "\n";
    }
};

// Derived class 2
class VegetableCrop : public Crop {
    int irrigationFrequency;

public:
    VegetableCrop(string n, double a, string s,
                  double y, int irrigation)
        : Crop(n, a, s, y) {
        if (irrigation <= 0)
            throw invalid_argument("Invalid irrigation frequency.");

        irrigationFrequency = irrigation;
    }

    // Overloaded constructor
    VegetableCrop(string n, double a, double y)
        : Crop(n, a, y) {
        irrigationFrequency = 3;
    }

    void display() const override {
        cout << "\n--- Vegetable Crop Details ---";
        Crop::display();
        cout << "Irrigation Frequency: "
             << irrigationFrequency << " times per week\n";
    }

    void giveAdvice() const override {
        cout << "Advice: Maintain proper soil moisture, "
             << "use organic manure and monitor pests regularly.\n";
    }

    void save(ofstream& file) const override {
        file << "VEGETABLE " << quoted(name) << " "
             << area << " " << quoted(season) << " "
             << yieldPerAcre << " "
             << irrigationFrequency << "\n";
    }
};

// Function to read positive numerical input
double readPositive(string message) {
    double value;

    while (true) {
        cout << message;

        if (cin >> value && value > 0)
            return value;

        cout << "Invalid input! Enter a positive number.\n";
        cin.clear();
        cin.ignore(numeric_limits<streamsize>::max(), '\n');
    }
}

// Save crop information
void saveData(const vector<unique_ptr<Crop>>& crops) {
    ofstream file("farmdata.txt");

    if (!file) {
        cout << "Unable to open file!\n";
        return;
    }

    for (const auto& c : crops)
        c->save(file);

    file.close();
    cout << "Crop data saved successfully!\n";
}

// Retrieve crop information
void loadData(vector<unique_ptr<Crop>>& crops) {
    ifstream file("farmdata.txt");

    if (!file) {
        cout << "No saved file found!\n";
        return;
    }

    vector<unique_ptr<Crop>> loaded;
    string type, name, season, fertilizer;
    double area, yield;
    int irrigation;

    try {
        while (file >> type) {
            if (type == "CEREAL") {
                file >> quoted(name) >> area
                     >> quoted(season) >> yield
                     >> quoted(fertilizer);

                loaded.push_back(make_unique<CerealCrop>(
                    name, area, season, yield, fertilizer));
            }
            else if (type == "VEGETABLE") {
                file >> quoted(name) >> area
                     >> quoted(season) >> yield
                     >> irrigation;

                loaded.push_back(make_unique<VegetableCrop>(
                    name, area, season, yield, irrigation));
            }
            else {
                throw runtime_error("Invalid file format.");
            }

            if (!file)
                throw runtime_error("Incomplete crop data.");
        }

        crops = move(loaded);
        cout << "Crop data retrieved successfully!\n";
    }
    catch (const exception& e) {
        cout << "File error: " << e.what() << endl;
    }

    file.close();
}

int main() {
    vector<unique_ptr<Crop>> crops;
    int choice;

    do {
        cout << "\n========== SMART FARM MANAGEMENT SYSTEM ==========\n";
        cout << "1. Add Cereal Crop\n";
        cout << "2. Add Vegetable Crop\n";
        cout << "3. Display All Crops and Advice\n";
        cout << "4. Compare Two Crops\n";
        cout << "5. Save Crop Data\n";
        cout << "6. Retrieve Crop Data\n";
        cout << "7. Display Total Crop Count\n";
        cout << "0. Exit\n";
        cout << "Enter your choice: ";

        if (!(cin >> choice)) {
            cout << "Invalid menu input!\n";
            cin.clear();
            cin.ignore(numeric_limits<streamsize>::max(), '\n');
            continue;
        }

        try {
            switch (choice) {

            case 1: {
                string name, season, fertilizer;
                double area, yield;

                cout << "Enter cereal crop name: ";
                cin >> name;

                area = readPositive("Enter area (acres): ");
                cout << "Enter season: ";
                cin >> season;

                yield = readPositive("Enter yield per acre: ");

                cout << "Enter fertilizer type: ";
                cin >> fertilizer;

                crops.push_back(make_unique<CerealCrop>(
                    name, area, season, yield, fertilizer));

                cout << "Cereal crop added successfully!\n";
                break;
            }

            case 2: {
                string name, season;
                double area, yield;
                int irrigation;

                cout << "Enter vegetable crop name: ";
                cin >> name;

                area = readPositive("Enter area (acres): ");
                cout << "Enter season: ";
                cin >> season;

                yield = readPositive("Enter yield per acre: ");

                cout << "Enter irrigation frequency per week: ";
                cin >> irrigation;

                crops.push_back(make_unique<VegetableCrop>(
                    name, area, season, yield, irrigation));

                cout << "Vegetable crop added successfully!\n";
                break;
            }

            case 3:
                if (crops.empty()) {
                    cout << "No crop records available.\n";
                    break;
                }

                for (const auto& c : crops) {
                    c->display();
                    c->giveAdvice();
                }
                break;

            case 4: {
                if (crops.size() < 2) {
                    cout << "Add at least two crops first.\n";
                    break;
                }

                int i, j;

                cout << "Enter first crop number (1 to "
                     << crops.size() << "): ";
                cin >> i;

                cout << "Enter second crop number (1 to "
                     << crops.size() << "): ";
                cin >> j;

                if (i < 1 || j < 1 ||
                    i > (int)crops.size() ||
                    j > (int)crops.size() || i == j) {
                    throw invalid_argument("Invalid crop selection.");
                }

                if (*crops[i - 1] > *crops[j - 1])
                    cout << "First crop has higher total yield.\n";
                else
                    cout << "Second crop has higher or equal yield.\n";

                break;
            }

            case 5:
                saveData(crops);
                break;

            case 6:
                loadData(crops);
                break;

            case 7:
                cout << "Total number of crops: "
                     << Crop::getCropCount() << endl;
                break;

            case 0:
                cout << "Exiting Smart Farm Management System.\n";
                break;

            default:
                cout << "Invalid choice! Try again.\n";
            }
        }
        catch (const exception& e) {
            cout << "Error: " << e.what() << endl;
        }

    } while (choice != 0);

    return 0;
}