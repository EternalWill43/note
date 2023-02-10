## SOLID Principles

*   [**SRP**](#single-responsibility-principle): The Single Responsibility Principle

    An active corollary to Conway’s law: The best structure for a software system is heavily influenced by the social structure of the organization that uses it so that each software module has one, and only one, reason to change.

*   [**OCP**](#open-closed-principle): The Open-Closed Principle

    Bertrand Meyer made this principle famous in the 1980s. The gist is that for software systems to be easy to change, they must be designed to allow the behavior of those systems to be changed by adding new code, rather than changing existing code.

*   **LSP**: The Liskov Substitution Principle

    Barbara Liskov’s famous definition of subtypes, from 1988. In short, this principle says that to build software systems from interchangeable parts, those parts must adhere to a contract that allows those parts to be substituted one for another.

*   **ISP**: The Interface Segregation Principle

    This principle advises software designers to avoid depending on things that they don't use.

*   **DIP**: The Dependency Inversion Principle

    The code that implements high-level policy should not depend on code that implements low-level details. Rather, details should depend on policies.


### Single Responsibility Principle

*A module should be responsible to one, and only one, actor.*

- Module: A source file.
- Actor: Single Person/Group/Stakeholder requiring the module to change

Canonical example is:

```c++
class Employee {
    //data
public:
    double calculateOvertime(int hours) {}
};
```

If there are multiple departments, and overtime is calculated differently, you don't want a situation where if you think you change the way overtime is calculated for one department, affects another department without them knowing about it.

In which case you might need to change the parameter list so everyone who calls it has to be much more explicit, but this can lead to a lot of coupling of logic and messy code.

For example:
```c++
double Employee::calculateOvertime(int hours, Department kDepartmentName) {
    if (kDepartmentName == kManagementStaff) {
        // do stuff
    } else if (kDepartmentName == kPayrollStaff) {
        // do other stuff
    }
}
```

If you don't change the API, it's possible that there are other modules referencing hours and payment, and they don't know anything about the changes being made.

Basically, even if it leads to some level of re-writing code (and some aditional boilerplate), you really want to de-couple code that can affect multiple actors into their own modules. 

Another solution is to use the facade pattern. The methods would belong to other, more complex classes, and your function calls would always remain the same. So perhaps other classes that take in the info and do the more complex and ugly code behind the facade interface, but that way if you have inter-team operability the way the API is called shoudn't change or affect people who don't want/need any changes made.

```c++
class OvertimeCalculator {
    //...
    ManagementOvertimeCalculator managementCalculator;
    ContractorOvertimeCalculator contractorCalculator;
public:
    //...
    double calculateOvertime(int hours, Department kDepartmentName) {
        if (kDepartmentName == kManagement) {
            return managaementCalculator.calculate(hours);
        } else if // etc
    }
}

// Facade Employee Class

class Employee {
    OvertimeCalculator overtimeCalculator;
    //...
public:
    double calculateOvertime(int hours, Department kDepartmentName) {
        return overtimeCalculator.calculateOvertime(hours, kDepartmentname);
    }
}

```

In a situation like that, where you know there is going to be similar functionality but might differ slightly depending on the actor who calls it, you want to keep the interface simple and unchanging, i.e., you only ever have to provide the hours and department, and the facade heirarchy will be able to change without you ever needing to alter that source file, so that way, if the Payroll Department overtime needs to change, only the class PayrollOvertimeCalculator needs to be altered.

Another example from Wikipedia:

```c++
struct CPU {
  void Freeze();
  void Jump(long position);
  void Execute();
};

struct HardDrive {
  char* Read(long lba, int size);
};

struct Memory {
  void Load(long position, char* data);
};

class ComputerFacade {
 public:
  void Start() {
    cpu_.Freeze();
    memory_.Load(kBootAddress, hard_drive_.Read(kBootSector, kSectorSize));
    cpu_.Jump(kBootAddress);
    cpu_.Execute();
  }

 private:
  CPU cpu_;
  Memory memory_;
  HardDrive hard_drive_;
};

int main() {
  ComputerFacade computer;
  computer.Start();
}
```

### Open Closed Principle

*A software artifact should be open for extension but closed for modification.*

Not extensible without modification:

```c++
enum class COLOR { RED, GREEN, BLUE };
enum class SIZE { SMALL, MEDIUM, LARGE };

struct Product {
    string  m_name;
    COLOR   m_color;
    SIZE    m_size;
};

using Items = vector<Product*>;
#define ALL(C)  begin(C), end(C)

struct ProductFilter {
    static Items by_color(Items items, const COLOR e_color) {
        Items result;
        for (auto &i : items)
            if (i->m_color == e_color)
                result.push_back(i);
        return result;
    }
    static Items by_size(Items items, const SIZE e_size) {
        Items result;
        for (auto &i : items)
            if (i->m_size == e_size)
                result.push_back(i);
        return result;
    }
    static Items by_size_and_color(Items items, const SIZE e_size, const COLOR e_color) {
        Items result;
        for (auto &i : items)
            if (i->m_size == e_size && i->m_color == e_color)
                result.push_back(i);
        return result;
    }
};

int main() {
    const Items all{
        new Product{"Apple", COLOR::GREEN, SIZE::SMALL},
        new Product{"Tree", COLOR::GREEN, SIZE::LARGE},
        new Product{"House", COLOR::BLUE, SIZE::LARGE},
    };

    for (auto &p : ProductFilter::by_color(all, COLOR::GREEN))
        cout << p->m_name << " is green\n";

    for (auto &p : ProductFilter::by_size_and_color(all, SIZE::LARGE, COLOR::GREEN))
        cout << p->m_name << " is green & large\n";

    return EXIT_SUCCESS;
}
/*
Apple is green
Tree is green
Tree is green & large
*/
```

Now extensible:

```cpp
template <typename T>
struct Specification {
    virtual ~Specification() = default;
    virtual bool is_satisfied(T *item) const = 0;
};

struct ColorSpecification : Specification<Product> {
    COLOR e_color;
    ColorSpecification(COLOR e_color) : e_color(e_color) {}
    bool is_satisfied(Product *item) const { return item->m_color == e_color; }
};

struct SizeSpecification : Specification<Product> {
    SIZE e_size;
    SizeSpecification(SIZE e_size) : e_size(e_size) {}
    bool is_satisfied(Product *item) const { return item->m_size == e_size; }
};

template <typename T>
struct Filter {
    virtual vector<T *> filter(vector<T *> items, const Specification<T> &spec) = 0;
};

struct BetterFilter : Filter<Product> {
    vector<Product *> filter(vector<Product *> items, const Specification<Product> &spec) {
        vector<Product *> result;
        for (auto &p : items)
            if (spec.is_satisfied(p))
                result.push_back(p);
        return result;
    }
};

// ------------------------------------------------------------------------------------------------
BetterFilter bf;
for (auto &x : bf.filter(all, ColorSpecification(COLOR::GREEN)))
    cout << x->m_name << " is green\n";

```

Combining filters:

```c++
template <typename T>
struct AndSpecification : Specification<T> {
    const Specification<T> &first;
    const Specification<T> &second;

    AndSpecification(const Specification<T> &first, const Specification<T> &second)
    : first(first), second(second) {}

    bool is_satisfied(T *item) const { 
        return first.is_satisfied(item) && second.is_satisfied(item); 
    }
};

template <typename T>
AndSpecification<T> operator&&(const Specification<T> &first, const Specification<T> &second) {
    return {first, second};
}

// -----------------------------------------------------------------------------------------------------

auto green_things = ColorSpecification{COLOR::GREEN};
auto large_things = SizeSpecification{SIZE::LARGE};

BetterFilter bf;
for (auto &x : bf.filter(all, green_things && large_things))
    cout << x->m_name << " is green and large\n";

// warning: the following will compile but will NOT work
// auto spec2 = SizeSpecification{SIZE::LARGE} &&
//              ColorSpecification{COLOR::BLUE};
```

