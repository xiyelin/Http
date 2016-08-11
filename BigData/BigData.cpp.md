### 模拟库中的atoi()函数处理各种非法/合法数据实现简单的大数加减乘除运算

--------------------------------------------------------

<br>

```cpp



		# include "BigData.h"
		
		BigData::BigData(INT64 value = 0xcccccccccccccccc)
			:_value(value)
		{
			INIT64ToString();
		}
		
		BigData::BigData(const char* pData)
		{
			//"12345678" "+" "+1234567"  "234567qwe"  "      "    "0000001234567" 
		 
			if (NULL == pData)
			{
				assert(false);
				return;
			}
		
			char* pStr = (char*)pData;
			char cSymbol = pData[0];
			if ('+' == cSymbol || '-' == cSymbol)
			{
				++pStr;
			}
			else if (cSymbol >= '0'&&cSymbol <= '9')
			{
				cSymbol = '+';
			}
			else
			{
				return;
			}
		
			while ('0' == (*pStr))
				++pStr;
		
		
			_strData.resize(strlen(pData) + 1);
			_strData[0] = cSymbol;
			_value = 0;
			int iCount = 1;
		
			while (*pStr >= '0' && *pStr <= '9')
			{
				_value = _value * 10 + (*pStr - '0');
				_strData[iCount++] = *pStr;
				++pStr;
			}
		
			_strData.resize(iCount);
			if (cSymbol == '-')
			{
				_value = 0 - _value;
			}
		}
		
		
		bool BigData::IsINIT64Overflow()const
		{
			string tmp("+9223372036854775807");
			if (_strData[0] == '-')
			{
				tmp = "-9223372036854775808";
			}
			
			if (_strData.size() < tmp.size())
			{
				return true;
			}
			else if (_strData.size() == tmp.size() && _strData <= tmp)
			{
				return true;
			}
			else
				return false;
		}
		
		void BigData::INIT64ToString()
		{
			//符号位
			char cSymbol = '+';
		
			if (_value < 0)
			{
				cSymbol = '-';
			}
		
			//获得的字符串为逆序
			INT64 tmp = _value;
			_strData.append(1, cSymbol);
			while (tmp)
			{
				int c = tmp % 10;
				if (c < 0)
				{
					c = 0 - c;
				}
				_strData.append(1, c + '0');
				tmp /= 10;
			}
		
			//逆置字符串
			char* pleft = (char*)_strData.c_str() + 1;
			char* pright = pleft + _strData.size() - 2;
		
			while (pleft < pright)
			{
				char ptmp = *pleft;
				*pleft++ = *pright;
				*pright-- = ptmp;
			}
		}
		
		ostream& operator<<(ostream& out, const BigData& bigdata)
		{
			if (bigdata.IsINIT64Overflow())
			{
				out << bigdata._value;
				return out;
			}
			else
			{
				char* pData = (char*)bigdata._strData.c_str();
				if ('+' == pData[0])
				{
					++pData;
				}
				out << pData;
				return out;
			}
		}
		
		BigData BigData::operator+(const BigData& bigdata)
		{
			if (IsINIT64Overflow() && bigdata.IsINIT64Overflow())
			{
				//+ -
				if (_strData[0] != bigdata._strData[0])
				{
					return BigData(_value + bigdata._value);
				}
				else
				{
					//10 -3 >= 8
					//-10 - (-3 ) = -7 > -6 -8
		
					if ((_value >= 0 && MAX_INT64 - _value >= bigdata._value) ||
						(_value < 0 && MIN_INT64 - _value <= bigdata._value))
					{
						return BigData(_value + bigdata._value);
					}
				}
			}
		
			//至少有一个溢出
			//结果溢出
			if (_strData[0] == bigdata._strData[0])
			{
				return BigData(Add(_strData, bigdata._strData).c_str());
			}
		
			return BigData(Sub(_strData, bigdata._strData).c_str());
		}
		
		string BigData::Add(string left, string right)
		{
			char cSymbol = '+';
			int iLSize = left.size();
			int iRSize = right.size();
		
			if (iLSize < iRSize)
			{
				swap(left, right);
				swap(iLSize, iRSize);
			}
		
			string sRet;
			sRet.resize(iLSize + 1);
			sRet[0] = left[0];
			char Step = 0;
		
			for (int iIdx = 1; iIdx < iLSize; ++iIdx)
			{
				char cRet = left[iLSize - iIdx] - '0' + Step;
		
				if (iIdx < iRSize)
				{
					cRet += (right[iRSize - iIdx] - '0');
				}
		
				sRet[iLSize - iIdx + 1] = cRet % 10 + '0';
				Step = cRet / 10;
			}
		
			sRet[1] = Step + '0';
		
			return "";
		}
		
		BigData BigData::operator-(const BigData& bigdata)
		{
			if (IsINIT64Overflow() && bigdata.IsINIT64Overflow())
			{
				return BigData(_value - bigdata._value);
			}
			else
			{
				// 10 + (-2) = 8
				if ((_value > 0 && MAX_INT64 + bigdata._value >= _value)||
					(_value < 0 && MIN_INT64 + bigdata._value <= _value))
				{
					return BigData(_value - bigdata._value);
				}
			}
		
			if (_strData[0] != bigdata._strData[0])
			{
				return BigData(Add(_strData, bigdata._strData).c_str());
			}
		
			return BigData(Sub(_strData, bigdata._strData).c_str());
		}
		
		string BigData::Sub(string left, string right)
		{
			int iLSize = left.size();
			int iRSize = right.size();
			char cSymbol = left[0];
		
			if (iLSize < iRSize || iLSize == iRSize && left < right)
			{
				swap(left, right);
				swap(iLSize, iRSize);
		
				if (cSymbol == '+')
				{
					cSymbol = '-';
				}
				else
				{
					cSymbol = '+';
				}
			}
		
			//保存结果
			string strRet;
			strRet.resize(left.size());
			strRet[0] = cSymbol;
		
			//从低往高  去left的每一位
			//从低往高  取right的每一位
			for (int Idx = 1; Idx < iLSize; ++Idx)
			{
				char cRet = left[iLSize - Idx];
				
				if (Idx < iRSize)
				{
					cRet -= (right[iRSize - Idx] - '0');
				}
		
				if (cRet < 0)
				{
					left[iRSize - Idx - 1] -= 1;
					cRet += 10;
				}
			}
		
			return "";
		}
		
		BigData BigData::operator*(const BigData& bigdata)
		{
			if (_value == 0 || bigdata._value == 0)
			{
				return BigData(INT64(0));
			}
			if (IsINIT64Overflow() && bigdata.IsINIT64Overflow())
			{
				if (_strData[0] == bigdata._strData[0])
				{
					if ((_value > 0 && MAX_INT64 / _value >= bigdata._value) ||
						(_value < 0 && MAX_INT64 / _value <= bigdata._value))
					{
						return BigData(_value * bigdata._value);
					}
				}
			}
			else
			{
				if ((_value > 0 && MIN_INT64 / _value <= bigdata._value) ||
					(_value < 0 && MIN_INT64 / _value >= bigdata._value))
				{
					return BigData(_value * bigdata._value);
				}
			}
		
		
			return BigData(Mul(_strData, bigdata._strData).c_str());
		}
		
		string BigData::Mul(string left, string right)
		{
			char cSymbol = '+';
			if (left[0] != right[0])
			{
				cSymbol = '-';
			}
			int iLSize = left.size();
			int iRSize = right.size();
		
			if (iLSize > iRSize)
			{
				swap(left, right);
				swap(iLSize, iRSize);
			}
		
			string sRet;
			/*sRet.resize(iLSize + iRSize - 1);*/
			sRet.assign(iLSize + iRSize - 1, '0');
			sRet[0] = cSymbol;
			int iDataLen = sRet.size();
			int ioffset = 0;
			//先取左边一个
			//和右边每一位相乘
			for (int iLIdx = 1; iLIdx < iLSize; ++iLIdx)
			{
				char cLeft = left[iLSize - iRSize] - '0';
				char c = left[iLSize - iLIdx];
				char cStep = 0;
		
				if (cLeft == 0)
				{
					++ioffset;
					continue;
				}
		
				for (int iRIdx = 1; iRIdx < iRSize; ++iRIdx)
				{
					char cRet = cLeft*(right[iRSize - iRIdx] - '0') + cStep;
					cRet += sRet[iDataLen - iRIdx - ioffset] - '0';
					sRet[iDataLen - iRIdx] += (cRet % 10) + '0';
					cStep = cRet / 10;
				}
		
				sRet[iDataLen - iRSize - ioffset] += cStep;
				++ioffset;
			}
		
			return sRet;
		}
		
		
		//除数不能为零		
		//如果两个都没有溢出  直接处
		
		//left	right
		//left < right -> 0
		//right = (+1)/(-1)
		//left == right
		//循环相减法  保证左边一定要比右边大  使用一个指针指向被除数的起始位置  通过指针的偏移来访问
		BigData BigData::operator / (const BigData& bigdata)
		{
			if ('0' == bigdata._strData[1])
			{
				assert(false);
		
			}
		
			if (IsINIT64Overflow() && bigdata.IsINIT64Overflow())
			{
				return BigData(_value / bigdata._value);
			}
		
			if ((_strData.size() < bigdata._strData.size()) ||
				(_strData.size() == bigdata._strData.size()) &&
				(strcmp(_strData.c_str() + 1, bigdata._strData.c_str() + 1) < 0))
			{
				return BigData(INT64(0));
			}
		
			if (bigdata._strData == "+1" || bigdata._strData == "-1")
			{
				string ret = _strData;
				if (_strData[0] != bigdata._strData[0])
				{
					ret[0] = '-';
				}
				return BigData(ret.c_str());
			}
		
			if (strcmp(_strData.c_str() + 1, bigdata._strData.c_str() + 1) == 0)
			{
				string ret = "+1";
				if (_strData[0] != bigdata._strData[0])
				{
					ret[0] = '-';
				}
				return BigData(ret.c_str());
		
			}
		
			return BigData(Div(_strData, bigdata._strData).c_str());
		}
		
		string BigData::Div(string left, string right)
		{
			string sRet;
			if (left[0] != right[0])
			{
				sRet[0] = '-';
			}
		
			char* pLeft = (char*)left.c_str() + 1;
			char* pRight = (char*)right.c_str() + 1;
			int DataLen = right.size() - 1;
			int LSize = left.size() - 1;
		
			for (int iIdx = 0; iIdx < left.size() - 1; )
			{
				if (*pLeft == '0')
				{
					sRet.append(1, '0');
					++pLeft;
					++iIdx;
					continue;
				}
		
				if (IsLeftStrBig(pLeft, DataLen, pRight, right.size() - 1))
				{
					sRet.append(1, '0');
					++DataLen;
					if (DataLen + iIdx > LSize)
					{
						break;
					}
				}
				else
				{
					//循环相减
					sRet.append(1, SubLoop(pLeft, DataLen, pRight, right.size() - 1));
					while (*pLeft == '0' && DataLen > 0)
					{
						++pLeft;
						++iIdx;
						--DataLen;
					}
		
					++DataLen;
					if (DataLen + iIdx > )
					{
						break;
					}
				}
			}
		
			return sRet;
		}
		
		bool BigData::IsLeftStrBig(const char* pLeft, int LSize, const char* pRight, int RSize)
		{
			if ((LSize > RSize) || LSize == RSize && strcmp(pLeft, pRight) >= 0)
			{
				return true;
			}
		
			return false;
		}
		
		char BigData::SubLoop(char* pLeft, int LSize, const char* pRight, int RSize)
		{
			assert(pLeft != NULL && pRight != NULL);
			char cRet = '0';
		
			while (true)
			{
				if (!IsLeftStrBig(pLeft, LSize, pRight, RSize))
					break;
		
				int LDataLen = LSize - 1;
				int RDataLen = RSize - 1;
				while (LDataLen > 0 && RDataLen > 0)
				{
					char ret = pLeft[LDataLen] - '0';
					ret -= pRight[RDataLen] - '0';
					if (ret < 0)
					{
						pLeft[LDataLen - 1] -= 1;
						ret += 10;
					}
		
					pLeft[LDataLen] = ret + '0';
					--LDataLen;
					--RDataLen;
				}
		
				while (*pLeft == '0')
				{
					++pLeft;
					--LSize;
				}
		
				++cRet;
			}
			
			return cRet;
		}

		
		
		
		
```
