# 1.制作字体时如何添加新的编码？

1.打开`A:\Tools\fontmake\fontripper.c`

```c

static void Init_Charset_Array()
{
	unsigned int Unicode = 0;
	int table_size = 0, i = 0;
	int len;

	for (Unicode = 0x20; Unicode <= 0x7F; Unicode++)
	{
		InsertChar(Unicode);
	}

	for (Unicode = 0xA1; Unicode <= 0xFF; Unicode++)
	{
		InsertChar(Unicode);
	}

	for (Unicode = 0x590; Unicode <= 0x77F; Unicode++)
	{
		InsertChar(Unicode);
	}

	InsertSort(s_range, s_Char_Sum);
}
```
此处的意思为添加的Unicode编码为`0x20~0x7F`, `0xA1~0xFF`, `0X590~0x77F`

```c
static FONT_STYLE fontlist[] = {
		{16, 0, GEN_COLOR_INDEX8, "./ttf/arial.ttf"},
		{18, 0, GEN_COLOR_INDEX8, "./ttf/arial.ttf"},
		{20, 0, GEN_COLOR_INDEX8, "./ttf/arial.ttf"},
		{22, 0, GEN_COLOR_INDEX8, "./ttf/arial.ttf"},
		{24, 0, GEN_COLOR_INDEX8, "./ttf/arial.ttf"},
};
```
此处的意思为`字体大小`，`是否粗体`，`多少个bit显示一个像素`，`字体`

2.在同级目录去执行`make clean && make`

3.执行`./fontripper`

4.将会生成`GenFont_16.c`, `GenFont_18.c`, `GenFont_20.c`, `GenFont_22.c`, `GenFont_24.c`,将每个文件中的两个数组拷贝至工程的`GenFont.c`。

# 2.如果UI显示了？应该如何修改

`F:\LocalProject\montage\Montage_Panaccess_Dvbc_sym2\MICO\prj\dvb\rosemary_demo\sources_pcam_dvbt2\MP_Graphic\ctrl\mp_ui_resourcemanager.c`文件中的`const MP_UINT8* STBGetString ( MP_INT32 nId )
`按照如下修改

```c
const MP_UINT8* STBGetString ( MP_INT32 nId )
{
	MP_INT32 tableLen = s_nCurStringTableLength;
	MP_INT32 ii = 0;

	static MP_UINT8 utf8_str_idx = 0;
	static MP_UINT8 utf8_str[10][1024];

	MP_UINT8 str_idx = utf8_str_idx;

	for(ii = 0; ii < tableLen; ii++)
	{
		if(s_pCurStringTable[ii].Id == nId)
		{
			//return (MP_UINT8*)s_pCurStringTable[ii].text;
			utf8_str[str_idx][0] = 0x15;
			mp_strncpy(utf8_str[str_idx]+1, s_pCurStringTable[ii].text, sizeof(utf8_str[str_idx])-1);
			utf8_str_idx = (str_idx+1)%10;
			return utf8_str[str_idx];
		}
	}
	return MP_NULL;
}
```

# 3.UBOOT