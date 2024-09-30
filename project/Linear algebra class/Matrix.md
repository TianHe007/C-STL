# 矩阵类

- 功能：
    - 矩阵变换分数数据类型使得精度丢失率极低
    - 加，减，数乘，矩阵相乘，转置，幂次，初等变换
    - 伴随矩阵，逆矩阵，矩阵行列式的值，后方增添/删除矩阵，矩阵的秩
    - 获取并输出齐次/非齐次线性方程组的解向量
    - 演示：矩阵输出，初等行变换后输出，解向量输出
    ![image](https://github.com/user-attachments/assets/1eb27a01-e6f8-43f7-a725-b10d83fd5de2)

    
- 实现矩阵类，如何适应不同数据类型？模板？
    - 对于矩阵的数据类型，要么整数要么浮点数，要么代数（不考虑）
    - 对于整数和浮点数，我们可以使用模板进行分类然后进行程序设计
    - 有没有一种办法可以同时兼容这两种情况呢？
- 实现分数类
    - 分数可以兼容整数和浮点数两类，并且分数不易出现精度丢失！
    - 分数类，成员：分母和分子，引入构造函数，设计两种构造函数，
        - 一种是通过分子和分母进行构造，默认情况为分子为0分母为1表示整数0，
        - 另一种是浮点构造，将其不断乘10化成整数，然后对于分母乘10，此分数表示的就是浮点数
        - 对于一些非法构造，进行异常抛出即可
        - 对于传入的分数，我们先对他执行化简，方便运算，`Simplify`函数为类函数，可供外部访问，先调用类内部函数`adjust`进行分数符号的确定，负号放到分子上方，然后通过`gcd`进行分数化简
        
        ```cpp
          	static void Simplify(Fraction &x) {
                adjust(x);
                if (x.nume == 0) {
                    x.deno = 1;
                    return;
                }
                int gcd = __gcd(std::abs(x.nume), std::abs(x.deno));
                x.nume /= gcd, x.deno /= gcd;
                return ;
            }
            
            static void adjust(Fraction &x) {
                bool f1 = x.nume < 0, f2 = x.deno < 0;
                if (f1 && f2) 
                    x.nume = std::abs(x.nume), x.deno = std::abs(x.deno);
                else if (f1 || f2) 
                    x.nume = -std::abs(x.nume), x.deno = std::abs(x.deno);
                return;
            }
        ```
        
        - 完善构造函数
        
        ```cpp
        class Fraction {
        public :
            const double eps = 1e-7;
            Fraction(int nume = 0, int deno = 1) : nume(nume), deno(deno) {
                if (!deno) 
                    throw std::runtime_error("The denominator is 0!");
                adjust(*this);
            }
            Fraction(double x) : nume(1), deno(1) {
                while (x - (int)x >= eps) { x *= 10, deno *= 10; }
                nume = (int)x;
                adjust(*this);
                return;
            }
            Fraction(const Fraction &obj) : nume(obj.nume), deno(obj.deno) {}
            ~Fraction() = default;
        ```
        
    - 实现四则运算以及比较规则
        - 首先需要实现分数通分函数，为类方法，传入两个分数进行通分，然后实现成员方法分数的绝对值以及分数的导数，这两个函数设计为**右值**
        
        ```cpp
         static void Same_Fraction_Deno(Fraction &a, Fraction &b) {
                int lcm = __lcm(a.deno, b.deno);
                a.nume *= lcm / a.deno;
                b.nume *= lcm / b.deno;
                a.deno = b.deno = lcm;
                return;
            }
           Fraction abs() {
                adjust(*this);
                Fraction ret(*this);
                ret.nume = std::abs(ret.nume);
                return ret;
            }
            Fraction backwards() {
                Fraction ret(*this);
                std::swap(ret.nume, ret.deno);
                adjust(ret);
                return ret;
            }
        ```
        
        - 准备工作完成后，实现分数的四则运算以及比较规则
        
        ```cpp
        Fraction &operator+=(const Fraction &obj) {
                Fraction temp(obj);
                Same_Fraction_Deno(*this, temp);
                this->nume += temp.nume;
                Simplify(*this);
                return *this;
            }
            Fraction operator+(const Fraction &obj) {  //调用+=实现+
                Fraction ret(*this);
                ret += obj;
                return ret;
            }
            Fraction &operator-=(const Fraction &obj) {
                Fraction temp(obj);
                Same_Fraction_Deno(*this, temp);
                this->nume -= temp.nume;
                Simplify(*this);
                return *this;
            }
            Fraction operator-(const Fraction &obj) {
                Fraction ret(*this);
                ret -= obj;
                return ret;
            }
            Fraction operator-() {
                Fraction ret(*this);
                adjust(ret);
                ret.nume = -ret.nume;
                return ret;
            }
            Fraction &operator*=(const Fraction &obj) {
                this->nume *= obj.nume;
                this->deno *= obj.deno;
                Simplify(*this);
                return *this;
            }
            Fraction operator*(const Fraction &obj) {
                Fraction ret(*this);
                ret *= obj;
                return ret;
            }
            Fraction &operator/=(const Fraction &obj) { 
                Fraction ret(obj);
                *this *= ret.backwards();
                Simplify(*this);
                return *this;
            }
            Fraction operator/(const Fraction &obj) {
                Fraction ret(*this);
                ret /= obj;
                return ret;
            }
            Fraction &operator=(const Fraction &obj) {
                this->nume = obj.nume;
                this->deno = obj.deno;
                return *this;
            }
            Fraction &operator++() {
                *this += 1;
                return *this;
            }
            Fraction operator++(int) {
                Fraction ret(*this);
                *this += 1;
                return ret;
            }
            Fraction &operator--() {
                *this -= 1;
                return *this;
            }
            Fraction operator--(int) {
                Fraction ret(*this);
                *this -= 1;
                return ret;
            }
            bool operator<(const Fraction &obj) { 
                Fraction temp(obj);
                Same_Fraction_Deno(*this, temp);
                return this->nume < temp.nume;
            }
            bool operator<=(const Fraction &obj) {
                Fraction temp(obj);
                Same_Fraction_Deno(*this, temp);
                return this->nume <= temp.nume;
            }
            //通过上面的规则来实现下面的规则
            bool operator>(const Fraction &obj) { return !(*this <= obj); }
            bool operator>=(const Fraction &obj) { return !(*this < obj); }
            bool operator==(const Fraction &obj) { return !(*this < obj) && !(*this > obj); }
            bool operator!=(const Fraction &obj) { return !(*this == obj); }
        ```
        
    - 接下来为分数的输出问题，肯定要转化为字符串形式进行输出，对于矩阵来说，每一个元素都是分数，那么每一个分数转化为字符串的长度不一样，输出肯定很难看，那么不妨确定所有元素中最大的长度Max_size，然后统一以这个长度进行输出
    - getsize获取当前分数的字符串长度，to_string根据maxsize进行转字符串
    
    ```cpp
     std::string to_string(int Max_size = 0) {
            Simplify(*this);
            std::string ret;
            if (deno == 1) ret = std::to_string(nume);
            else ret = std::to_string(nume) + "/" + std::to_string(deno);
            int diff = Max_size - ret.size();
            if (diff <= 0) return ret;
            ret = std::string((diff + 1) / 2, ' ') + ret + std::string((diff) / 2, ' ');
            return ret;
        }
        size_t get_Size() {
            Simplify(*this);
            return (nume == 0 ? 1 : (int)log10(std::abs(nume)) + 1) + (int)log10(deno) + 1 + (nume < 0) + 1;
        }
    ```
    
    - 有了转字符串函数，就可以重载osteam进行cout输出了
    
    ```cpp
     friend std::ostream &operator<<(std::ostream &out, const Fraction &obj) {
            Fraction temp(obj);
            out << temp.to_string();
            return out;
        }
    ```
    
    - 再补充一些小细节方面的函数，分数类自此完成：完整代码：
    
    ```cpp
    class Fraction {
    public :
        const double eps = 1e-7;
        Fraction(int nume = 0, int deno = 1) : nume(nume), deno(deno) {
            if (!deno) 
                throw std::runtime_error("The denominator is 0!");
            Simplify(*this);
            return;
        }
        Fraction(double x) : nume(1), deno(1) {
            while (x - (int)x >= eps) { x *= 10, deno *= 10; }
            nume = (int)x;
            Simplify(*this);
            return;
        }
        Fraction(const Fraction &obj) : nume(obj.nume), deno(obj.deno) {}
        ~Fraction() = default;
        Fraction &operator+=(const Fraction &obj) {
            Fraction temp(obj);
            Same_Fraction_Deno(*this, temp);
            this->nume += temp.nume;
            Simplify(*this);
            return *this;
        }
        Fraction operator+(const Fraction &obj) {
            Fraction ret(*this);
            ret += obj;
            return ret;
        }
        Fraction &operator-=(const Fraction &obj) {
            Fraction temp(obj);
            Same_Fraction_Deno(*this, temp);
            this->nume -= temp.nume;
            Simplify(*this);
            return *this;
        }
        Fraction operator-(const Fraction &obj) {
            Fraction ret(*this);
            ret -= obj;
            return ret;
        }
        Fraction operator-() {
            Fraction ret(*this);
            adjust(ret);
            ret.nume = -ret.nume;
            return ret;
        }
        Fraction &operator*=(const Fraction &obj) {
            this->nume *= obj.nume;
            this->deno *= obj.deno;
            Simplify(*this);
            return *this;
        }
        Fraction operator*(const Fraction &obj) {
            Fraction ret(*this);
            ret *= obj;
            return ret;
        }
        Fraction &operator/=(const Fraction &obj) {  //Fraction &opeartor/=(Fraction this, const Fraciont &obj)
            Fraction ret(obj);
            *this *= ret.backwards();
            Simplify(*this);
            return *this;
        }
        Fraction operator/(const Fraction &obj) {
            Fraction ret(*this);
            ret /= obj;
            return ret;
        }
        Fraction &operator=(const Fraction &obj) {
            this->nume = obj.nume;
            this->deno = obj.deno;
            return *this;
        }
        Fraction &operator++() {
            *this += 1;
            return *this;
        }
        Fraction operator++(int) {
            Fraction ret(*this);
            *this += 1;
            return ret;
        }
        Fraction &operator--() {
            *this -= 1;
            return *this;
        }
        Fraction operator--(int) {
            Fraction ret(*this);
            *this -= 1;
            return ret;
        }
        bool operator<(const Fraction &obj) { 
            Fraction temp(obj);
            Same_Fraction_Deno(*this, temp);
            return this->nume < temp.nume;
        }
        bool operator<=(const Fraction &obj) {
            Fraction temp(obj);
            Same_Fraction_Deno(*this, temp);
            return this->nume <= temp.nume;
        }
        bool operator>(const Fraction &obj) { return !(*this <= obj); }
        bool operator>=(const Fraction &obj) { return !(*this < obj); }
        bool operator==(const Fraction &obj) { return !(*this < obj) && !(*this > obj); }
        bool operator!=(const Fraction &obj) { return !(*this == obj); }
        int get_num() { return nume; }
        int get_deno() { return deno; }
        double decimul() { return 1.0 * nume / deno; }
        std::string to_string(int Max_size = 0) {
            Simplify(*this);
            std::string ret;
            if (deno == 1) ret = std::to_string(nume);
            else ret = std::to_string(nume) + "/" + std::to_string(deno);
            int diff = Max_size - ret.size();
            if (diff <= 0) return ret;
            ret = std::string((diff + 1) / 2, ' ') + ret + std::string((diff) / 2, ' ');
            return ret;
        }
        size_t get_Size() {
            Simplify(*this);
            return (nume == 0 ? 1 : (int)log10(std::abs(nume)) + 1) + (int)log10(deno) + 1 + (nume < 0) + 1;
        }
        static void Simplify(Fraction &x) {
            adjust(x);
            if (x.nume == 0) {
                x.deno = 1;
                return;
            }
            int gcd = __gcd(std::abs(x.nume), std::abs(x.deno));
            x.nume /= gcd, x.deno /= gcd;
            return ;
        }
        static void Same_Fraction_Deno(Fraction &a, Fraction &b) {
            int lcm = __lcm(a.deno, b.deno);
            a.nume *= lcm / a.deno;
            b.nume *= lcm / b.deno;
            a.deno = b.deno = lcm;
            return;
        }
        Fraction abs() {
            adjust(*this);
            Fraction ret(*this);
            ret.nume = std::abs(ret.nume);
            return ret;
        }
        Fraction backwards() {
            Fraction ret(*this);
            std::swap(ret.nume, ret.deno);
            adjust(ret);
            return ret;
        }
        friend std::ostream &operator<<(std::ostream &out, const Fraction &obj) {
            Fraction temp(obj);
            out << temp.to_string();
            return out;
        }
    private :
        int nume;
        int deno;
        static int __gcd(int a, int b) {
            return b ? __gcd(b, a % b) : a;
        }
        static int __lcm(int a, int b) {
            return a / __gcd(a, b) * b;
        }
        static void adjust(Fraction &x) {
            bool f1 = x.nume < 0, f2 = x.deno < 0;
            if (f1 && f2) 
                x.nume = std::abs(x.nume), x.deno = std::abs(x.deno);
            else if (f1 || f2) 
                x.nume = -std::abs(x.nume), x.deno = std::abs(x.deno);
            return;
        }
    };
    ```
    
- 下面着重实现矩阵类，存储矩阵我们用两层vector进行存储，类型为分数类型
- 设计构造函数，我们设计两种构造函数：

![image](https://github.com/user-attachments/assets/d369850e-5000-486c-a307-851054836314)


- 上述两种构造函数功能一致，均生成2行三列的矩阵
- 第一种使用初始化列表`std::initializer_list<T>` 将元素赋值然后推算出行数和列数即可
    - 如果发现某一行的元素和其他行的元素不一样，那么直接异常处理
- 第二种使用行数和列数以及一维列表实现，较为简单
- 构造函数分为左值构造和右值构造（移动构造），对于移动构造直接将地址搬过来即可
- self_col的作用见后面解释
- 由于构造函数我们使用了左值构造和右值构造，那么相应的`operator=`操作符我们是不是也要重新实现一遍呢？
    - 我们直接基于拷贝构造进行赋值运算，称为**原地构造，**保证效率的同时可以节省代码量

```cpp
class Matrix {
public :
    Matrix(std::initializer_list<std::initializer_list<Fraction>> __data)
    : row(__data.size()), col(__data.begin()->size()), self_col(col), data(__data.begin(), __data.end()) {
        for (auto it = __data.begin(); it != __data.end(); it++) {
            if (it->size() != col) 
                throw std::runtime_error("illegal intput initializer_list, columns is different");
        }
        return;
    }
    Matrix() : row(1), col(1), self_col(col), data(row, std::vector<Fraction>(col)) {}
    Matrix(size_t row, size_t col, std::vector<Fraction> __data = {})
    : row(row), col(col), self_col(col), data(row, std::vector<Fraction>(col)) {
        if (row <= 0 || col <= 0) {
            throw std::runtime_error("illegal input : row <= 0 or col <= 0!");
        }
        for (int i = 0, k = 0; i < row; i++) {
            for (int j = 0; j < col; j++) {
                if (k >= __data.size()) return;
                data[i][j] = __data[k++];
            }
        }
        return;
    }
    Matrix(const Matrix &obj) : row(obj.row), col(obj.col), self_col(obj.self_col), data(obj.data) {}
    Matrix(const Matrix &&obj) : row(obj.row), col(obj.col), self_col(obj.self_col), data(std::move(obj.data)) {}
    ~Matrix() = default;
    Matrix &operator=(const Matrix &obj) {
        new(this) Matrix(obj);
        return *this;
    }
    Matrix &operator=(const Matrix &&obj) {
        new(this) Matrix(std::move(obj));
        return *this;
    }
}
```

- 随后就是`+,-,*,T,pow,[]`的实现：
    - 首先由于幂次操作需要单位矩阵，我们先实现单位矩阵的逻辑：
    - 成员方法`SetEye()`将当前矩阵置为单位阵，行列不同报错，实际就是通过类方法`E()`返回一个row行row列的单位阵，然后通过移动构造赋值给当前矩阵再返回即可，很明显返回左值
    
    ```cpp
    Matrix &SetEye() {  //将当前矩阵置为单位阵
            if (row != col) 
                throw std::runtime_error("this Matrix can't be Eye");
            *this = std::move(E(row));
            return *this;
        }
    static Matrix E(size_t row = 1) {  //类方法，返回row=col的单位阵
            Matrix res;
            std::vector<std::vector<Fraction>> data(row, std::vector<Fraction>(row));
            for (int i = 0; i < row; i++) data[i][i] = 1;
            res.row = res.col = res.self_col = row;
            res.data = std::move(data);
            return res;
        }
    ```
    
    - 随后实现加法，减法，乘法，转置，幂次，快捷访问：
    
    ```cpp
    //加减法都需要矩阵行数列数相同，不同直接抛出异常
    Matrix &operator+=(const Matrix &obj) {
            if (row != obj.row || col != obj.col) 
                throw std::runtime_error("Only two matrices of the same type can perform addition operations!");
            for (int i = 0; i < row; i++) {
                for (int j = 0; j < col; j++) {
                    data[i][j] += obj.data[i][j];
                }
            }
            return *this;
        }
        Matrix operator+(const Matrix &obj) {
            Matrix ret(*this);
            ret += obj;
            return ret;
        }
        Matrix &operator-=(const Matrix &obj) {
            if (row != obj.row || col != obj.col) 
                throw std::runtime_error("Only two identical matrices can be subtracted");
            for (int i = 0; i < row; i++) {
                for (int j = 0; j < col; j++) {
                    data[i][j] -= obj.data[i][j];
                }
            }
            return *this;
        }
        Matrix operator-(const Matrix &obj) {
            Matrix ret(*this);
            ret -= obj;
            return ret;
        }
        Matrix &operator*=(const Fraction &x) {
            for (int i = 0; i < row; i++) {
                for (int j = 0; j < col; j++) {
                    data[i][j] *= x;
                }
            }
            return *this;
        }
        Matrix operator*(const Fraction &x) {
            Matrix ret(*this);
            ret *= x;
            return ret;
        }
        //数乘以矩阵的重载，这里没有*=
        friend Matrix operator*(const Fraction &x, const Matrix &obj) {
            Matrix ret(obj);
            ret *= x;
            return ret;
        }
        //矩阵乘法需要当前列数等于乘的矩阵的行数否则抛出异常
        Matrix operator*=(const Matrix &obj) {
            if (col != obj.row) 
                throw std::runtime_error("Not meeting the rules of matrix multiplication");
            Matrix ret(row, obj.col);
            for (int i = 0; i < row; i++) {
                for (int j = 0; j < obj.col; j++) {
                    for (int k = 0; k < obj.row; k++) {
                        ret.data[i][j] += data[i][k] * obj.data[k][j];
                    }
                }
            }
            *this = std::move(ret);
            return *this;
        }
        Matrix operator*(const Matrix &obj) {
            Matrix ret(*this);
            ret *= obj;
            return ret;
        }
        std::vector<Fraction> &operator[](size_t indx) {  //重载[]，使得能够直接访问矩阵元素
            return data[indx];  //由于vector中已经重载过了[],因此我们只需要重载一个即可
        }
        Matrix pow(int b) {  //快速幂计算矩阵的幂
        //计算幂次返回右值，不能影响当前this，所以需要两个局部变量
            Matrix res = std::move(E(row)), temp(*this);
            while (b) {
                if (b & 1) res *= temp;
                temp *= temp;
                b >>= 1;
            }
            return res;
        }
        Matrix &Transposition() {
            std::swap(this->col, this->row);
            std::vector<std::vector<Fraction>> __data(this->row, std::vector<Fraction>(this->col));
            for (int i = 0; i < this->row; i++) {
                for (int j = 0; j < this->col; j++) {
                    __data[i][j] = data[j][i];
                }
            }
            this->data = std::move(__data);
            return *this;
        }
    ```
    
- 随后输出我们的矩阵看一下吧：
    - 首先转为字符串形式，然后输出
    - 次数就要用到分数输出的maxsize了，首先遍历矩阵所有元素，获取最长长度然后统一进行输出：
    
    ```cpp
    std::string to_string() {
            size_t Max_size = get_Max_size(*this);
            std::string res;
            for (int j = 0; j < data.size(); j++) {
                auto &x = data[j];
                res += "[ ";
                for (int i = 0; i < x.size(); i++) {
                    auto &y = x[i];
                    res += y.to_string(Max_size);
                    if (i != x.size() - 1) res += ", ";
                }
                res += " ]";
                if (j != data.size() - 1) res += "\n";
            }
            return res;
        }
        
        static size_t get_Max_size(Matrix &obj) {
            size_t Max_size = 1;
            for (auto &x : obj.data) {
                for (auto &y : x) Max_size = std::max(Max_size, y.get_Size());
            }
            return Max_size;
        }
        
        friend std::ostream &operator<<(std::ostream &out, const Matrix &obj) {
            Matrix temp(obj);
            out << temp.to_string() << '\n';
            return out;
        }
    ```
    
- 矩阵的初等变换：
    - 为方便求矩阵的行列式的值，再进行变换的时候存储操作产生的值即可，将矩阵化为最简的时候，此值就是行列式的值，由于获取需要传入引用参数，但是我们不希望用户传入参数，因此采用封装的思想：
    - 初等变换步骤：
        - 找到该列绝对值最大的元素，然后将这一行换到除固定行外的最上面，然后将这一行的第一个元素变成1
        - 随后用这一行的这一列，将其他行的这一列的元素变成0，其他列的元素跟着变
        - 随后往回推，化为最简
        - 由于增广矩阵包含系数列，而反推时不能包含系数列，因此引入selfcol表示不含系数列的列数，而col表示矩阵中所有的列数
    
    ```cpp
     Matrix &Elementary_Transformation() {  //矩阵的初等变化
            Fraction res = 1;
            return __Elementary_Transformation(res);  //调用类内部的方法，res传入引用为对应行列式的操作的值，res就是矩阵的行列式的值
        }
        
        Matrix &__Elementary_Transformation(Fraction &cnt) { //矩阵初等行变换化为最简的方法
            for (int c = 0, r = 0; c < col; c++) {
                int t = r;  //记录列绝对值最大行数
                if (t >= row) break;
                for (int i = r + 1; i < row; i++) {
                    if (data[i][c].abs() > data[t][c].abs()) t = i;
                }
                if (data[t][c] == 0) continue;
    	           //交换到最上面
                for (int j = c; j < col; j++) std::swap(data[r][j], data[t][j]);
                if (r != t) cnt = -cnt;  //交换两行两列行列式变号
                cnt *= data[r][c];  //矩阵首元素变成1除的元素对应行列式乘对应元素
                //倒着除，如果要正着除需要另外设置一个变量，因为除了值就发生变换
                for (int j = col - 1; j >= c; j--) data[r][j] /= data[r][c];
                //遍历它下面的所有行
                for (int i = r + 1; i < row; i++) {
                    if (data[i][c] == 0) continue;  //首元素为0则不管
                    //否则这一列从后往前减去对应的值
                    for (int j = col - 1; j >= c; j--) 
                        data[i][j] -= data[i][c] * data[r][j];
                }
                r++;
            }
            //此时为下三角形式，随后我们通过逆推将行首元素为1的列的其他列变成0
            //即可化为行最简型
            for (int i = row - 1; i >= 1; i--) {
                int t = -1;
                for (int j = 0; j < self_col; j++) {
                    if (data[i][j] == 1) {
                        t = j;
                        break;
                    }
                }
                if (t == -1) continue;
                if (data[i - 1][t] == 0) continue;
                for (int r = i - 1; r >= 0; r--) {
                    for (int j = col - 1; j >= t; j--) {
                        data[r][j] -= data[r][t] * data[i][j];
                    }
                }
            }
            return *this;
        }
    ```
    
- 通过矩阵的初等变换，可以很容易求出矩阵的逆矩阵，伴随矩阵，矩阵的秩，矩阵行列式的值
    
    ```cpp
    Matrix Inverse_Matrix() {  //矩阵的逆矩阵
            if (this->Determinant() == 0)   //矩阵的行列式值为0说明没有逆矩阵
                throw std::runtime_error("this Matrix don't have Inverse Matrix");
            Matrix res(*this);
            if (res.col == res.self_col) res.push_back();
            res.Elementary_Transformation();  //使用的是初等变化法求逆矩阵，首先需要在res后面添加单位阵
            for (int i = 0; i < row; i++) {
                for (int j = self_col - 1; j >= 0; j--) {
                    std::swap(res.data[i][j], res.data[i][j + self_col]);
                    res.data[i].pop_back();
                }
            }
            res.col = res.self_col;
            return res;
        }
        Matrix Adjoint_Matrix() {  //矩阵的伴随，通过逆矩阵和行列式求得
            return this->Inverse_Matrix() * this->Determinant();
        }
        Fraction Determinant() {  //矩阵行列式的值
            if (row != col)   //求行列式首先row要等于col
                throw std::runtime_error("not a determinant!");
            Matrix res(*this);
            Fraction cnt = 1;
            res.__Elementary_Transformation(cnt);
            return cnt;
        }
        size_t rank() {  //矩阵的秩，初等行变换求
            Matrix res(*this);
            res.Elementary_Transformation();
            size_t cnt = 0;
            for (int i = 0; i < res.row; i++) {
                for (int j = 0; j < res.col; j++) {
                    if (res.data[i][j] == 1) {
                        cnt += 1;
                        break;
                    }
                }
            }
            return cnt;
        }
    ```
    
- 求齐次/非齐次线性方程组解向量：
    - 由于齐次后面再加一列全0元素就可以直接使用非齐次的方法求，因此调用齐次的时候先push一列全0元素然后再调用非齐次方法
    - 对应push方法，默认想当前矩阵后面加与当前矩阵相同行的单位矩阵， 否则传入要push的矩阵，pop需要传入需要pop的列数
    
    ```cpp
    Matrix &push_back() {  //左值，默认添加自己的单位阵
            push_back(E(row));  //调用类内的函数重载方法，push一个矩阵
            return *this;
        }
        Matrix &push_back(const Matrix &obj) {  //push一个矩阵到this的末尾
            if (row != obj.row)   //如果两个矩阵行不相等则无法push
                throw std::runtime_error("The number of rows in two matrices does not match");
            for (int i = 0; i < row; i++) {
                for (int j = self_col; j < self_col + obj.col; j++) {
                    data[i].emplace_back(obj.data[i][j - self_col]);
                }
            }
            col += obj.col;
            return *this;
        }
        Matrix &pop_back(size_t cnt) {  //矩阵末尾弹出cnt列
            for (int i = 0; i < row; i++) {
                for (int j = 0; j < cnt; j++) {
                    data[i].pop_back();
                }
            }
            this->col -= cnt;
            return *this;
        }
    ```
    
    - 齐次与非齐次，统一调用类内的方法：
    
    ```cpp
    std::string Homogeneous_Linear_Equations() {  //返回矩阵齐次线性方程组的解
            Matrix res(*this);
            res.push_back(Matrix(res.row, 1));  //push一个全0列然后返回矩阵非齐次线性方程组的解向量即可
            return __Non_Homogeneous_Linear_Equations(res);
        }
        std::string Non_Homogeneous_Linear_Equations() {  //返回矩阵非齐次线性方程组的解向量
            return __Non_Homogeneous_Linear_Equations(*this);
        }
    ```
    
    - 假设a为不含系数列的矩阵，b为含系数列的矩阵
    - 当rank(a) < rank(b)的时候，无解
    - 否则使用新矩阵res表示解向量构成的矩阵
        - 首先`res`的行数是`a`的列数，也就是未知数的个数，`res`的列数是无法确定解的未知数的个数，也就是`a.col - rank(a) + 1`列
        - 将a和b都进行初等行变换之后，从左向右找第一个非零元素也就是1，然后这个1所在的列col就是未知数$x_{col}$，然后遍历它后面的元素，将后面的非零元素填入到res矩阵中
        - 特设set集合，插入未知数1，2，3，…..，如果遍历过程中找到1元素，那么这个列对应的未知数可以由其他未知数唯一确定，将其删去，set集合中剩下的就是无法唯一确定，需要设立c1，c2…..变量的未知数
        - 将set集合剩下的元素在res矩阵中用1表示，填入完毕后res就为解向量构成的矩阵，然后通过字符串方法输出即可
        - 代码解释：
        
        ```
         static std::string __Non_Homogeneous_Linear_Equations(const Matrix &obj) {  //进行非齐次线性方程组解的计算
                Matrix a(obj), b(obj);
                a.self_col = a.col - 1;
                a.pop_back(1);
                size_t ra = a.rank();
                size_t rb = b.rank();
                if (ra < rb) return "No solutions";  //矩阵和增广矩阵秩不等则无解
                a.Elementary_Transformation();
                b.Elementary_Transformation();
                std::set<int> s;  //记录最后需要代数替换的未知数为哪些
                for (int i = 0; i < a.col; i++) s.insert(i);
                Matrix res(a.col, a.col - ra + 1);
                for (int i = 0; i < ra; i++) {
                    int indx = i;
                    for (int j = i; j < b.col; j++) {
                        if (b.data[i][j] == 1) {
                            indx = j;  //indx为一行中第一个非零元素
                            break;
                        }
                    }
                    s.erase(indx);  //indx可以被唯一确定
                    //如果为非系数列，那么就是相反数，否则为原数
                    //此处为从后向前遍历，目的是为了方便写入无法被确定的未知数的值
                    for (int j = b.col - 1, cnt = res.col - 1; j > indx; j--) {
                        if (j != a.col && a.data[i][j] == 0) continue;
                        if (j == a.col) res.data[indx][cnt] = b.data[i][j];
                        else res.data[indx][cnt] = -b.data[i][j];
                        cnt--;
                    }
                }
                //将无法唯一确定的未知数填充为1，后续再向量左边乘以一个未知数表示
                int cnt = 0;
                for (auto &x : s) res.data[x][cnt++] = 1;
                //下面为res矩阵转化为字符串形式且含有未知数的形式的输出
                std::string ans("The solution vector of this linear equation system is:\n");
                size_t row = res.row, col = res.col, middle = row / 2, Max_size = get_Max_size(res);  //下面为解向量的输出部分
                for (int i = 0; i < row; i++) {
                    ans += "[x(" + std::to_string(i + 1) + ")]" +  (i == middle ? std::string(5, ' ') : std::string(9, ' '));
                    if (i == middle) ans[ans.size() - 3] = '=';  //中间行需要额外输入=，c(x)
                    for (int j = 0; j < col; j++) {
                        if (i == middle && j != col - 1) ans += "c(" + std::to_string(j + 1) + ")";
                        ans += "[" + res.data[i][j].to_string(Max_size) + "]";
                        if (i != middle) ans += std::string(9, ' ');
                        else if (i == middle && j < col - 2) ans += std::string("  +  ");
                        else if (j < col - 1) ans += std::string("    +    ");
                        else {
                            ans += "   {";
                            for (int k = 0; k < col - 1; k++) 
                                ans += "c(" + std::to_string(k + 1) + ")" + (k == col - 2 ? " " : ", ");
                            ans += "-> R}";
                        }
                        if (j == col - 1) ans += '\n';
                    }
                }
                return ans;
            }
        ```
        
- 至此已经实现了矩阵中基础的方法，如果需要输出变换过程则只需要再步骤中添加输出即可，总代码如下：

```cpp
#include <iostream>
#include <vector>
#include <numeric>
#include <string>
#include <cmath>
#include <exception>
#include <map>
#include <set>
#define debug(T) std::cout << T << std::endl;

class Fraction {
public :
    const double eps = 1e-7;
    Fraction(int nume = 0, int deno = 1) : nume(nume), deno(deno) {
        if (!deno) 
            throw std::runtime_error("The denominator is 0!");
        Simplify(*this);
    }
    Fraction(double x) : nume(1), deno(1) {
        while (x - (int)x >= eps) { x *= 10, deno *= 10; }
        nume = (int)x;
        Simplify(*this);
        return;
    }
    Fraction(const Fraction &obj) : nume(obj.nume), deno(obj.deno) {}
    Fraction &operator+=(const Fraction &obj) {
        Fraction temp(obj);
        Same_Fraction_Deno(*this, temp);
        this->nume += temp.nume;
        Simplify(*this);
        return *this;
    }
    Fraction operator+(const Fraction &obj) {
        Fraction ret(*this);
        ret += obj;
        return ret;
    }
    Fraction &operator-=(const Fraction &obj) {
        Fraction temp(obj);
        Same_Fraction_Deno(*this, temp);
        this->nume -= temp.nume;
        Simplify(*this);
        return *this;
    }
    Fraction operator-(const Fraction &obj) {
        Fraction ret(*this);
        ret -= obj;
        return ret;
    }
    Fraction operator-() {
        Fraction ret(*this);
        adjust(ret);
        ret.nume = -ret.nume;
        return ret;
    }
    Fraction &operator*=(const Fraction &obj) {
        this->nume *= obj.nume;
        this->deno *= obj.deno;
        Simplify(*this);
        return *this;
    }
    Fraction operator*(const Fraction &obj) {
        Fraction ret(*this);
        ret *= obj;
        return ret;
    }
    Fraction &operator/=(const Fraction &obj) {  //Fraction &opeartor/=(Fraction this, const Fraciont &obj)
        Fraction ret(obj);
        *this *= ret.backwards();
        Simplify(*this);
        return *this;
    }
    Fraction operator/(const Fraction &obj) {
        Fraction ret(*this);
        ret /= obj;
        return ret;
    }
    Fraction &operator=(const Fraction &obj) {
        this->nume = obj.nume;
        this->deno = obj.deno;
        return *this;
    }
    Fraction &operator++() {
        *this += 1;
        return *this;
    }
    Fraction operator++(int) {
        Fraction ret(*this);
        *this += 1;
        return ret;
    }
    Fraction &operator--() {
        *this -= 1;
        return *this;
    }
    Fraction operator--(int) {
        Fraction ret(*this);
        *this -= 1;
        return ret;
    }
    bool operator<(const Fraction &obj) { 
        Fraction temp(obj);
        Same_Fraction_Deno(*this, temp);
        return this->nume < temp.nume;
    }
    bool operator<=(const Fraction &obj) {
        Fraction temp(obj);
        Same_Fraction_Deno(*this, temp);
        return this->nume <= temp.nume;
    }
    bool operator>(const Fraction &obj) { return !(*this <= obj); }
    bool operator>=(const Fraction &obj) { return !(*this < obj); }
    bool operator==(const Fraction &obj) { return !(*this < obj) && !(*this > obj); }
    bool operator!=(const Fraction &obj) { return !(*this == obj); }
    int get_num() { return nume; }
    int get_deno() { return deno; }
    double decimul() { return 1.0 * nume / deno; }  //分数化小数
    std::string to_string(int Max_size = 0) {
        Simplify(*this);
        std::string ret;
        if (deno == 1) ret = std::to_string(nume);
        else ret = std::to_string(nume) + "/" + std::to_string(deno);
        int diff = Max_size - ret.size();
        if (diff <= 0) return ret;
        ret = std::string((diff + 1) / 2, ' ') + ret + std::string((diff) / 2, ' ');
        return ret;
    }
    size_t get_Size() {   //获取分数转化为字符串所占大小
        Simplify(*this);
        return (nume == 0 ? 1 : (int)log10(std::abs(nume)) + 1) + (int)log10(deno) + 1 + (nume < 0) + 1;
    }
    static void Simplify(Fraction &x) {  //分数化简
        adjust(x);
        if (x.nume == 0) {
            x.deno = 1;
            return;
        }
        int gcd = __gcd(std::abs(x.nume), std::abs(x.deno));
        x.nume /= gcd, x.deno /= gcd;
        return ;
    }
    static void Same_Fraction_Deno(Fraction &a, Fraction &b) {   //分数通分
        int lcm = __lcm(a.deno, b.deno);
        a.nume *= lcm / a.deno;
        b.nume *= lcm / b.deno;
        a.deno = b.deno = lcm;
        return;
    }
    Fraction abs() {   //分数绝对值，返回右值
        adjust(*this);
        Fraction ret(*this);
        ret.nume = std::abs(ret.nume);
        return ret;
    }
    Fraction backwards() {  //分数倒数，返回右值
        Fraction ret(*this);
        std::swap(ret.nume, ret.deno);
        adjust(ret);
        return ret;
    }
    friend std::ostream &operator<<(std::ostream &out, const Fraction &obj) {
        Fraction temp(obj);
        out << temp.to_string();
        return out;
    }
private :
    int nume;
    int deno;
    static int __gcd(int a, int b) {
        return b ? __gcd(b, a % b) : a;
    }
    static int __lcm(int a, int b) {
        return a / __gcd(a, b) * b;
    }
    static void adjust(Fraction &x) {  //调节分子分母正负号，如果分母为负则调整为分子为负，方面转化字符串
        bool f1 = x.nume < 0, f2 = x.deno < 0;
        if (f1 && f2) 
            x.nume = std::abs(x.nume), x.deno = std::abs(x.deno);
        else if (f1 || f2) 
            x.nume = -std::abs(x.nume), x.deno = std::abs(x.deno);
        return;
    }
};

class Matrix {
public :
    Matrix(std::initializer_list<std::initializer_list<Fraction>> __data)  //初始化列表构造
    : row(__data.size()), col(__data.begin()->size()), self_col(col), data(__data.begin(), __data.end()) {
        for (auto it = __data.begin(); it != __data.end(); it++) {
            if (it->size() != col)   //防止初始化列表列数不统一
                throw std::runtime_error("illegal intput initializer_list, columns is different");
        }
        return;
    }
    Matrix() : row(1), col(1), self_col(col), data(row, std::vector<Fraction>(col)) {}  //行，列，初始化列表构造
    Matrix(size_t row, size_t col, std::vector<Fraction> __data = {})
    : row(row), col(col), self_col(col), data(row, std::vector<Fraction>(col)) {
        if (row <= 0 || col <= 0) {
            throw std::runtime_error("illegal input : row <= 0 or col <= 0!");
        }
        for (int i = 0, k = 0; i < row; i++) {
            for (int j = 0; j < col; j++) {
                if (k >= __data.size()) return;
                data[i][j] = __data[k++];
            }
        }
        return;
    }
    //左值拷贝，右值拷贝
    Matrix(const Matrix &obj) : row(obj.row), col(obj.col), self_col(obj.self_col), data(obj.data) {}
    Matrix(const Matrix &&obj) : row(obj.row), col(obj.col), self_col(obj.self_col), data(std::move(obj.data)) {}
    Matrix &operator=(const Matrix &obj) {
        new(this) Matrix(obj);
        return *this;
    }
    Matrix &operator=(const Matrix &&obj) {
        new(this) Matrix(std::move(obj));
        return *this;
    }
    Matrix &operator+=(const Matrix &obj) {
        if (row != obj.row || col != obj.col) 
            throw std::runtime_error("Only two matrices of the same type can perform addition operations!");
        for (int i = 0; i < row; i++) {
            for (int j = 0; j < col; j++) {
                data[i][j] += obj.data[i][j];
            }
        }
        return *this;
    }
    Matrix operator+(const Matrix &obj) {
        Matrix ret(*this);
        ret += obj;
        return ret;
    }
    Matrix &operator-=(const Matrix &obj) {
        if (row != obj.row || col != obj.col) 
            throw std::runtime_error("Only two identical matrices can be subtracted");
        for (int i = 0; i < row; i++) {
            for (int j = 0; j < col; j++) {
                data[i][j] -= obj.data[i][j];
            }
        }
        return *this;
    }
    Matrix operator-(const Matrix &obj) {
        Matrix ret(*this);
        ret -= obj;
        return ret;
    }
    Matrix &operator*=(const Fraction &x) {
        for (int i = 0; i < row; i++) {
            for (int j = 0; j < col; j++) {
                data[i][j] *= x;
            }
        }
        return *this;
    }
    Matrix operator*(const Fraction &x) {
        Matrix ret(*this);
        ret *= x;
        return ret;
    }
    friend Matrix operator*(const Fraction &x, const Matrix &obj) {
        Matrix ret(obj);
        ret *= x;
        return ret;
    }
    Matrix operator*=(const Matrix &obj) {
        if (col != obj.row) 
            throw std::runtime_error("Not meeting the rules of matrix multiplication");
        Matrix ret(row, obj.col);
        for (int i = 0; i < row; i++) {
            for (int j = 0; j < obj.col; j++) {
                for (int k = 0; k < obj.row; k++) {
                    ret.data[i][j] += data[i][k] * obj.data[k][j];
                }
            }
        }
        *this = std::move(ret);
        return *this;
    }
    Matrix operator*(const Matrix &obj) {
        Matrix ret(*this);
        ret *= obj;
        return ret;
    }
    std::vector<Fraction> &operator[](size_t indx) {  //重载[]，使得能够直接访问矩阵元素
        return data[indx];
    }
    Matrix pow(int b) {  //快速幂计算矩阵的幂
        Matrix res = std::move(E(row)), temp(*this);
        while (b) {
            if (b & 1) res *= temp;
            temp *= temp;
            b >>= 1;
        }
        return res;
    }
    std::string to_string() {  //将矩阵转化为字符串输出
        size_t Max_size = get_Max_size(*this);
        std::string res;
        for (int j = 0; j < data.size(); j++) {
            auto &x = data[j];
            res += "[ ";
            for (int i = 0; i < x.size(); i++) {
                auto &y = x[i];
                res += y.to_string(Max_size);
                if (i != x.size() - 1) res += ", ";
            }
            res += " ]";
            if (j != data.size() - 1) res += "\n";
        }
        return res;
    }
    Matrix &SetEye() {  //将当前矩阵置为单位阵
        if (row != col) 
            throw std::runtime_error("this Matrix can't be Eye");
        *this = std::move(E(row));
        return *this;
    }
    Matrix &Transposition() {  //矩阵的转置
        std::swap(this->col, this->row);
        std::vector<std::vector<Fraction>> __data(this->row, std::vector<Fraction>(this->col));
        for (int i = 0; i < this->row; i++) {
            for (int j = 0; j < this->col; j++) {
                __data[i][j] = data[j][i];
            }
        }
        this->data = std::move(__data);
        return *this;
    }
    Matrix &Elementary_Transformation() {  //矩阵的初等变化
        Fraction res = 1;
        return __Elementary_Transformation(res);  //调用类内的方法，res传入引用为对应行列式的操作的值，res就是矩阵的行列式的值
    }
    Matrix Inverse_Matrix() {  //矩阵的逆矩阵
        if (this->Determinant() == 0)   //矩阵的行列式值为0说明没有逆矩阵
            throw std::runtime_error("this Matrix don't have Inverse Matrix");
        Matrix res(*this);
        if (res.col == res.self_col) res.push_back();
        res.Elementary_Transformation();  //使用的是初等变化法求逆矩阵，首先需要在res后面添加单位阵
        for (int i = 0; i < row; i++) {
            for (int j = self_col - 1; j >= 0; j--) {
                std::swap(res.data[i][j], res.data[i][j + self_col]);
                res.data[i].pop_back();
            }
        }
        res.col = res.self_col;
        return res;
    }
    Matrix Adjoint_Matrix() {  //矩阵的伴随，通过逆矩阵和行列式求得
        return this->Inverse_Matrix() * this->Determinant();
    }
    Matrix &push_back() {  //左值，默认添加自己的单位阵
        push_back(E(row));  //调用类内的函数重载方法，push一个矩阵
        return *this;
    }
    Matrix &push_back(const Matrix &obj) {  //push一个矩阵到this的末尾
        if (row != obj.row)   //如果两个矩阵行不相等则无法push
            throw std::runtime_error("The number of rows in two matrices does not match");
        for (int i = 0; i < row; i++) {
            for (int j = self_col; j < self_col + obj.col; j++) {
                data[i].emplace_back(obj.data[i][j - self_col]);
            }
        }
        col += obj.col;
        return *this;
    }
    Matrix &pop_back(size_t cnt) {  //矩阵末尾弹出cnt列
        for (int i = 0; i < row; i++) {
            for (int j = 0; j < cnt; j++) {
                data[i].pop_back();
            }
        }
        this->col -= cnt;
        return *this;
    }
    Fraction Determinant() {  //矩阵行列式的值
        if (row != col)   //求行列式首先row要等于col
            throw std::runtime_error("not a determinant!");
        Matrix res(*this);
        Fraction cnt = 1;
        res.__Elementary_Transformation(cnt);
        return cnt;
    }
    size_t rank() {  //矩阵的秩，初等行变换求
        Matrix res(*this);
        res.Elementary_Transformation();
        size_t cnt = 0;
        for (int i = 0; i < res.row; i++) {
            for (int j = 0; j < res.col; j++) {
                if (res.data[i][j] == 1) {
                    cnt += 1;
                    break;
                }
            }
        }
        return cnt;
    }
    size_t cols() { return col; }
    size_t rows() { return row; }
    std::string Homogeneous_Linear_Equations() {  //返回矩阵齐次线性方程组的解
        Matrix res(*this);
        res.push_back(Matrix(res.row, 1));  //push一个全0列然后返回矩阵非齐次线性方程组的解向量即可
        return __Non_Homogeneous_Linear_Equations(res);
    }
    std::string Non_Homogeneous_Linear_Equations() {  //返回矩阵非齐次线性方程组的解向量
        return __Non_Homogeneous_Linear_Equations(*this);
    }
    static Matrix E(size_t row = 1) {  //类方法，返回row=col的单位阵
        Matrix res;
        std::vector<std::vector<Fraction>> data(row, std::vector<Fraction>(row));
        for (int i = 0; i < row; i++) data[i][i] = 1;
        res.row = res.col = res.self_col = row;
        res.data = std::move(data);
        return res;
    }
    friend std::ostream &operator<<(std::ostream &out, const Matrix &obj) {
        Matrix temp(obj);
        out << temp.to_string() << '\n';
        return out;
    }
private :
    size_t row, col, self_col;
    std::vector<std::vector<Fraction>> data;
    Matrix &__Elementary_Transformation(Fraction &cnt) { //矩阵初等行变换化为最简的方法
        for (int c = 0, r = 0; c < col; c++) {
            int t = r;
            if (t >= row) break;
            for (int i = r + 1; i < row; i++) {
                if (data[i][c].abs() > data[t][c].abs()) t = i;
            }
            if (data[t][c] == 0) continue;
            for (int j = c; j < col; j++) std::swap(data[r][j], data[t][j]);
            if (r != t) cnt = -cnt;
            cnt *= data[r][c];
            for (int j = col - 1; j >= c; j--) data[r][j] /= data[r][c];
            for (int i = r + 1; i < row; i++) {
                if (data[i][c] == 0) continue;
                for (int j = col - 1; j >= c; j--) 
                    data[i][j] -= data[i][c] * data[r][j];
            }
            r++;
        }
        for (int i = row - 1; i >= 1; i--) {
            int t = -1;
            for (int j = 0; j < self_col; j++) {
                if (data[i][j] == 1) {
                    t = j;
                    break;
                }
            }
            if (t == -1) continue;
            if (data[i - 1][t] == 0) continue;
            for (int r = i - 1; r >= 0; r--) {
                for (int j = col - 1; j >= t; j--) {
                    data[r][j] -= data[r][t] * data[i][j];
                }
            }
        }
        return *this;
    }
    static size_t get_Max_size(Matrix &obj) {  //获取矩阵中所有分数转为字符串所占用的最大长度
        size_t Max_size = 1;
        for (auto &x : obj.data) {
            for (auto &y : x) Max_size = std::max(Max_size, y.get_Size());
        }
        return Max_size;
    }
    static std::string __Non_Homogeneous_Linear_Equations(const Matrix &obj) {  //进行非齐次线性方程组解的计算
        Matrix a(obj), b(obj);
        a.self_col = a.col - 1;
        a.pop_back(1);
        size_t ra = a.rank();
        size_t rb = b.rank();
        if (ra < rb) return "No solutions";  //矩阵和增广矩阵秩不等则无解
        a.Elementary_Transformation();
        b.Elementary_Transformation();
        std::set<int> s;  //记录最后需要代数替换的未知数为哪些
        for (int i = 0; i < a.col; i++) s.insert(i);
        Matrix res(a.col, a.col - ra + 1);
        for (int i = 0; i < ra; i++) {
            int indx = i;
            for (int j = i; j < b.col; j++) {
                if (b.data[i][j] == 1) {
                    indx = j;
                    break;
                }
            }
            s.erase(indx);
            for (int j = b.col - 1, cnt = res.col - 1; j > indx; j--) {
                if (j != a.col && a.data[i][j] == 0) continue;
                if (j == a.col) res.data[indx][cnt] = b.data[i][j];
                else res.data[indx][cnt] = -b.data[i][j];
                cnt--;
            }
        }
        int cnt = 0;
        for (auto &x : s) res.data[x][cnt++] = 1;
        std::string ans("The solution vector of this linear equation system is:\n");
        size_t row = res.row, col = res.col, middle = row / 2, Max_size = get_Max_size(res);  //下面为解向量的输出部分
        for (int i = 0; i < row; i++) {
            ans += "[x(" + std::to_string(i + 1) + ")]" +  (i == middle ? std::string(5, ' ') : std::string(9, ' '));
            if (i == middle) ans[ans.size() - 3] = '=';
            for (int j = 0; j < col; j++) {
                if (i == middle && j != col - 1) ans += "c(" + std::to_string(j + 1) + ")";
                ans += "[" + res.data[i][j].to_string(Max_size) + "]";
                if (i != middle) ans += std::string(9, ' ');
                else if (i == middle && j < col - 2) ans += std::string("  +  ");
                else if (j < col - 1) ans += std::string("    +    ");
                else {
                    ans += "   {";
                    for (int k = 0; k < col - 1; k++) 
                        ans += "c(" + std::to_string(k + 1) + ")" + (k == col - 2 ? " " : ", ");
                    ans += "-> R}";
                }
                if (j == col - 1) ans += '\n';
            }
        }
        return ans;
    }
};

int main() {
    //教科书测试矩阵，结果一致
    Matrix A(3, 3, {1, 0, 1, 2, 1, 0, -3, 2, -5});
    Matrix B(3, 3, {1, 2, 3, 2, 2, 1, 3, 4, 3});
    Matrix C(4, 4, {1, 2, 1, 3, 4, -1, -5, -6, 1, -3, -4, -7, 2, 1, -1, 0});
    Matrix D(3, 3, {1, 2, 1, 3, 4, 2, 1, 2, 2});
    Matrix E(3, 3, {1, 2, 3, 2, 3, -5, 4, 7, 1});
    Matrix F(5, 5, {2, -1, 0, 3, -2, 0, 3, 1, -2, 5, 0, 0, 0, 4, -3, 0, 0, 0, 0, 0});
    Matrix G(3, 3, {4, 2, 3, 1, 1, 0, -1, 2, 3});
    Matrix H(3, 3, {1, 1, -1, -1, 1, 1, 1, -1, 1});
    Matrix I(3, 3, {1, 0, 1, 2, 1, 0, -3, 2, -5});
    Matrix J(3, 4, {1, 2, 2, 1, 2, 1, -2, -2, 1, -1, -4, -3});
    Matrix K(2, 3, {1, 2, -1, 2, 4, 7});
    Matrix L(3, 5, {1, 1, -3, -1, 1, 3, -1, -3, 4, 4, 1, 5, -9, -8, 0});
    Matrix M(4, 5, {2, -1, -1, 1, 2, 1, 1, -2, 1, 4, 4, -6, 2, -2, 4, 3, 6, -9, 7, 9});
    Matrix N(3, 4, {1, 1, -2, -3, 0, -3, 3, 6, 0, 0, 0, 0});
    Matrix O(3, 4, {1, 2, 3, -1, 3, 6, -1, -3, 5, 10, 1, -5});
    Matrix P(4, 4, {2, 3, 1, 4, 1, -2, 4, -5, 3, 8, -2, 13, 4, -1, 9, -6});
    Matrix R(3, 4, {1, 2, 3, 4, 2, 2, 3, 4, 3, 2, 3, 4});
    Matrix S(3, 4, {1, 2, 2, 1, 0, 1, 1, 0, 1, 2, -1, -1});
    Matrix T(4, 4, {1, 1, 1, 1, 1, 2, -1, 0, 2, 1, 4, 3, 2, 3, 0, 1});
    Matrix P113_2(3, 4, {1, 2, 1, -1, 3, 6, -1, -3, 5, 10, 1, -5});
    return 0;
}
```
