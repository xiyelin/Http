```cpp

		
		# ifndef BIG_DATA_H
		# define IG_DATA_H
		
		
		# include <iostream>
		# include <string>
		# include <assert.h>
		using namespace std;
		
		typedef long long INT64;
		# define UN_INIT 0xcccccccccccccccc
		# define MAX_INT64 0x7FFFFFFFFFFFFFFF
		# define MIN_INT64 0x8000000000000000
		
		class BigData
		{
			friend ostream& operator<<(ostream& out, const BigData& bigdata);
		public:
			BigData(INT64 value);
			BigData(const char* pData);
		
		public:
			BigData operator+(const BigData& bigdata);
			BigData operator-(const BigData& bigdata);
			BigData operator*(const BigData& bigdata);
			BigData operator/(const BigData& bigdata);
		private:
			bool IsINIT64Overflow()const;
			void INIT64ToString();
			bool IsLeftStrBig(const char* pLeft, int LSize, const char* pRight, int RSize);
			char SubLoop(char* pLeft, int LSize, const char* pRight, int RSize);
			string Add(string left, string right);
			string Sub(string left, string right);
			string Mul(string left, string right);
			string Div(string left, string right);
		
		private:
			INT64 _value;
			string _strData;
		};
		
		
		
		
		#endif
		
		
		
		
		
```
