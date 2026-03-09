#include <iostream>
#include <vector>
#include <string>
#include <algorithm>
#include <cmath>
#include <iomanip>
#include <functional>
#include <cstdint>

#ifndef M_PI
#define M_PI 3.14159265358979323846
#endif

class BigInt {
private:
    using InternalType = uint64_t;
    static constexpr InternalType BASE = 1000000000;
    std::vector<InternalType> data;
    bool negative = false;

    void trim() {
        while (data.size() > 1 && data.back() == 0) data.pop_back();
        if (data.size() == 1 && data[0] == 0) negative = false;
    }

public:
    BigInt(long long v = 0) {
        if (v < 0) { negative = true; v = -v; }
        if (v == 0) data.push_back(0);
        while (v > 0) {
            data.push_back(v % BASE);
            v /= BASE;
        }
    }

    BigInt(const std::string& s) {
        if (s.empty()) { data.push_back(0); return; }
        std::string src = s;
        if (src[0] == '-') { negative = true; src = src.substr(1); }
        for (int i = (int)src.size(); i > 0; i -= 9) {
            if (i < 9) data.push_back(std::stoull(src.substr(0, i)));
            else data.push_back(std::stoull(src.substr(i - 9, 9)));
        }
        trim();
    }

    bool operator<(const BigInt& other) const {
        if (negative != other.negative) return negative;
        if (data.size() != other.data.size()) return (data.size() < other.data.size()) ^ negative;
        for (int i = (int)data.size() - 1; i >= 0; --i) {
            if (data[i] != other.data[i]) return (data[i] < other.data[i]) ^ negative;
        }
        return false;
    }

    BigInt operator-() const {
        BigInt res = *this;
        if (!(data.size() == 1 && data[0] == 0)) res.negative = !negative;
        return res;
    }

    BigInt operator+(const BigInt& other) const {
        if (negative == other.negative) {
            BigInt res = *this;
            InternalType carry = 0;
            for (size_t i = 0; i < std::max(res.data.size(), other.data.size()) || carry; ++i) {
                if (i == res.data.size()) res.data.push_back(0);
                InternalType cur = carry + res.data[i] + (i < other.data.size() ? other.data[i] : 0);
                res.data[i] = cur % BASE;
                carry = cur / BASE;
            }
            return res;
        }
        return *this - (-other);
    }

    BigInt operator-(const BigInt& other) const {
        if (negative != other.negative) return *this + (-other);
        if (abs_less(other)) {
            BigInt res = other - *this;
            res.negative = !negative;
            return res;
        }
        BigInt res = *this;
        InternalType carry = 0;
        for (size_t i = 0; i < other.data.size() || carry; ++i) {
            long long cur = (long long)res.data[i] - carry - (i < other.data.size() ? other.data[i] : 0);
            carry = (cur < 0);
            if (carry) cur += BASE;
            res.data[i] = (InternalType)cur;
        }
        res.trim();
        return res;
    }

    BigInt operator*(const BigInt& other) const {
        BigInt res;
        res.data.resize(data.size() + other.data.size(), 0);
        for (size_t i = 0; i < data.size(); ++i) {
            InternalType carry = 0;
            for (size_t j = 0; j < other.data.size() || carry; ++j) {
                unsigned __int128 cur = (unsigned __int128)res.data[i + j] + 
                                        (unsigned __int128)data[i] * (j < other.data.size() ? other.data[j] : 0) + carry;
                res.data[i + j] = (InternalType)(cur % BASE);
                carry = (InternalType)(cur / BASE);
            }
        }
        res.negative = negative != other.negative;
        res.trim();
        return res;
    }

    friend std::ostream& operator<<(std::ostream& os, const BigInt& bi) {
        if (bi.negative) os << '-';
        os << bi.data.back();
        for (int i = (int)bi.data.size() - 2; i >= 0; --i)
            os << std::setfill('0') << std::setw(9) << bi.data[i];
        return os;
    }

private:
    bool abs_less(const BigInt& other) const {
        if (data.size() != other.data.size()) return data.size() < other.data.size();
        for (int i = (int)data.size() - 1; i >= 0; --i)
            if (data[i] != other.data[i]) return data[i] < other.data[i];
        return false;
    }
};

namespace MathEngine {
    double derivative(std::function<double(double)> f, double x) {
        double h = 1e-7;
        return (f(x + h) - f(x - h)) / (2.0 * h);
    }

    double integrate(std::function<double(double)> f, double a, double b) {
        int n = 10000;
        double h = (b - a) / n;
        double s = f(a) + f(b);
        for (int i = 1; i < n; i++) s += (i % 2 == 0 ? 2 : 4) * f(a + i * h);
        return s * h / 3.0;
    }
}

void showMenu() {
    std::cout << "\n------------------------------------" << std::endl;
    std::cout << "1. Addition (Huge Numbers)" << std::endl;
    std::cout << "2. Subtraction (Huge Numbers)" << std::endl;
    std::cout << "3. Multiplication (Huge Numbers)" << std::endl;
    std::cout << "4. Trigonometry (sin/cos/tan)" << std::endl;
    std::cout << "5. Numerical Derivative (f(x)=x^2)" << std::endl;
    std::cout << "6. Definite Integral (f(x)=x^2, [0,1])" << std::endl;
    std::cout << "7. Exit" << std::endl;
    std::cout << "------------------------------------" << std::endl;
    std::cout << "Enter Your Choice (1-7): ";
}

int main() {
    int choice;
    std::string s1, s2;
    double val;

    while (true) {
        showMenu();
        if (!(std::cin >> choice)) {
            std::cin.clear();
            std::cin.ignore(1000, '\n');
            continue;
        }

        if (choice == 7) break;

        switch (choice) {
            case 1:
                std::cout << "Enter two huge numbers: ";
                std::cin >> s1 >> s2;
                std::cout << "Result = " << BigInt(s1) + BigInt(s2) << std::endl;
                break;
            case 2:
                std::cout << "Enter two huge numbers: ";
                std::cin >> s1 >> s2;
                std::cout << "Result = " << BigInt(s1) - BigInt(s2) << std::endl;
                break;
            case 3:
                std::cout << "Enter two huge numbers: ";
                std::cin >> s1 >> s2;
                std::cout << "Result = " << BigInt(s1) * BigInt(s2) << std::endl;
                break;
            case 4:
                std::cout << "Enter angle in radians: ";
                std::cin >> val;
                std::cout << "sin: " << std::sin(val) << "\ncos: " << std::cos(val) << "\ntan: " << std::tan(val) << std::endl;
                break;
            case 5:
                std::cout << "Calculating f'(x) for f(x)=x^2. Enter x: ";
                std::cin >> val;
                std::cout << "Result = " << MathEngine::derivative([](double x){return x*x;}, val) << std::endl;
                break;
            case 6:
                std::cout << "Integral of x^2 from 0 to 1: ";
                std::cout << "Result = " << MathEngine::integrate([](double x){return x*x;}, 0.0, 1.0) << std::endl;
                break;
            default:
                std::cout << "Invalid Choice!" << std::endl;
        }
    }
    return 0;
}
