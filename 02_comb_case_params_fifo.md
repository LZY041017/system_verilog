# 鎵规 02锛歚always_comb`銆乣case`銆佸弬鏁般€佹暟缁勪笌绠€鍗?FIFO

鏉ユ簮锛歚training/02`銆乣training/05`銆乣training/07` 涓殑绠€鍖?鏀瑰啓鐗囨銆傜洰鏍囦笉鏄竴涓嬪瓙鍐欏嚭 FIFO锛岃€屾槸閫愪釜鐪嬫噦鍏剁粍鎴愰儴鍒嗐€?
---

## 01锝滅粍鍚堥€昏緫鐨勯粯璁よ祴鍊?
### 浠ｇ爜鐗囨

```systemverilog
always_comb begin
    y = 8'h00;
    if (select)
        y = a;
end
```

### 浣犵殑鍒ゆ柇

1. `select=0` 鏃?`y` 鏄粈涔堬紵
2. 杩欐鏄惁闇€瑕佹椂閽燂紵
3. 涓轰粈涔堢涓€琛屽厛缁?`y` 涓€涓粯璁ゅ€硷紵

<details>
<summary>鍙傝€冨垎鏋?/summary>

杩欐槸缁勫悎閫昏緫锛宍select=0` 鏃惰緭鍑?0锛宍select=1` 鏃惰緭鍑?`a`銆傞粯璁よ祴鍊肩‘淇濇墍鏈夎矾寰勯兘缁?`y` 璧嬪€硷紱鑻ユ病鏈夊畠锛宍select=0` 鏃?`y` 闇€瑕佲€滆浣忔棫鍊尖€濓紝缁煎悎鏃跺彲鑳芥帹鏂攣瀛樺櫒銆?
</details>

---

## 02锝滅粍鍚堥€昏緫涓殑閿佸瓨鍣ㄩ櫡闃?
### 浠ｇ爜鐗囨

```systemverilog
always_comb begin
    if (enable)
        y = a + b;
end
```

### 浣犵殑鍒ゆ柇

1. `enable=0` 鏃讹紝纭欢闇€瑕佹€庢牱寰楀埌 `y`锛?2. 杩欐涓庝笂涓€娈电殑鍏抽敭宸紓鏄粈涔堬紵

<details>
<summary>鍙傝€冨垎鏋?/summary>

`enable=0` 鏃舵病鏈変换浣曡祴鍊硷紝`y` 鍙兘淇濈暀鏃у€硷紝鍥犳浼氭帹鏂攣瀛樺櫒锛堟垨琚?lint 鎶ラ敊锛夈€傜粍鍚堟ā鍧楅€氬父涓嶅笇鏈涢殣钘忓瓨鍌ㄧ姸鎬侊紝搴斿湪寮€澶村啓榛樿鍊硷紝鎴栨槑纭啓鍑?`else`銆?
</details>

---

## 03锝渀always_comb` 涓?`assign`

### 浠ｇ爜鐗囨

```systemverilog
assign sum0 = a + b;

always_comb begin
    sum1 = a + b;
end
```

### 浣犵殑鍒ゆ柇

1. 涓よ€呭湪杩欐绠€鍗曚唬鐮佷腑鍔熻兘鏄惁鐩稿悓锛?2. 浣曟椂鏇撮€傚悎 `assign`锛熶綍鏃舵洿閫傚悎 `always_comb`锛?
<details>
<summary>鍙傝€冨垎鏋?/summary>

杩欓噷鍔熻兘鐩稿悓锛岄兘鏄粍鍚堥€昏緫銆傚崟涓€佺洿鎺ョ殑琛ㄨ揪寮忛€氬父鐢?`assign` 鏇寸畝娲侊紱瀛樺湪澶氫釜涓存椂鍙橀噺銆佹潯浠跺垎鏀垨 `case` 鏃讹紝`always_comb` 鏇存竻鏅般€備笉瑕佷负鍚屼竴涓俊鍙峰悓鏃跺啓 `assign` 鍜?`always_comb`锛岄偅浼氬舰鎴愬涓┍鍔ㄣ€?
</details>

---

## 04锝滃畬鏁寸殑 `if / else`

### 浠ｇ爜鐗囨

```systemverilog
always_comb begin
    if (signed_mode)
        result = $signed(a) + $signed(b);
    else
        result = a + b;
end
```

### 浣犵殑鍒ゆ柇

1. 涓轰粈涔堣繖閲屼笉闇€瑕侀澶栭粯璁よ祴鍊硷紵
2. `signed_mode` 鏄€滅‖浠堕厤缃綅鈥濊繕鏄€滅紪璇戦€夐」鈥濓紵

<details>
<summary>鍙傝€冨垎鏋?/summary>

涓や釜鍒嗘敮閮借鐩栦簡 `result`锛屽洜姝や笉闇€瑕侀粯璁ゅ€笺€俙signed_mode` 鏄緭鍏ヤ俊鍙凤紝缁煎悎鍚庡搴旇繍琛屾椂鍙敼鍙樼殑纭欢鎺у埗浣嶏紱瀹冧笉鏄紪璇戞湡鐨勫弬鏁般€傛敞鎰忥細鍗充娇浣跨敤 `$signed`锛宍result` 鏈韩鐨勪綅瀹戒粛浼氬奖鍝嶆槸鍚︽埅鏂€?
</details>

---

## 05锝滃湴鍧€璇戠爜 `case`

### 浠ｇ爜鐗囨

```systemverilog
always_comb begin
    case (rd_addr)
        4'h0:    rd_data = control;
        4'h4:    rd_data = scale;
        default: rd_data = '0;
    endcase
end
```

### 浣犵殑鍒ゆ柇

1. `rd_addr=4'h8` 鏃惰緭鍑轰粈涔堬紵
2. `default` 鍦ㄧ‖浠惰璁′腑涓轰粈涔堥噸瑕侊紵
3. 杩欐浠ｈ〃瀵勫瓨鍣ㄨ鍙栬繕鏄瘎瀛樺櫒鍐欏叆锛?
<details>
<summary>鍙傝€冨垎鏋?/summary>

鍦板潃 8 涓嶅尮閰嶅墠涓ら」锛屽洜姝よ緭鍑哄叏 0銆俙default` 浣挎湭鏄犲皠鍦板潃鏈夌‘瀹氳涓猴紝骞堕伩鍏嶇粍鍚堥€昏緫婕忚祴鍊笺€傚畠鏄粍鍚堣鍙栭€昏緫锛涚湡姝ｆ敼鍙?`control/scale` 鐨勫啓鍏ヤ細鍦?`always_ff` 涓彂鐢熴€?
</details>

---

## 06锝渀case` 娌℃湁 `default` 浼氭€庢牱

### 浠ｇ爜鐗囨

```systemverilog
always_comb begin
    case (opcode)
        2'b00: result = a + b;
        2'b01: result = a - b;
    endcase
end
```

### 浣犵殑鍒ゆ柇

1. `opcode=2'b10` 鏃跺彲鑳芥€庢牱锛?2. 涓ょ淇鏂瑰紡鏄粈涔堬紵

<details>
<summary>鍙傝€冨垎鏋?/summary>

褰?`opcode` 涓?10 鎴?11锛宍result` 娌℃湁璧嬪€硷紝浼氭帹鏂攣瀛樺櫒銆備慨澶嶆柟寮忎竴锛氬湪 `always_comb` 寮€澶寸粰 `result` 榛樿鍊硷紱鏂瑰紡浜岋細娣诲姞 `default: result = '0;`銆傚伐绋嬩笂甯稿悓鏃朵娇鐢ㄩ粯璁ゅ€煎拰 `default`锛屼互鎻愰珮鍙鎬с€?
</details>

---

## 07锝滃弬鏁版敼鍙樼殑鏄€滅敓鎴愮殑纭欢鈥?
### 浠ｇ爜鐗囨

```systemverilog
module configurable_counter #(
    parameter int Width = 4
) (
    output logic [Width-1:0] count
);
```

### 浣犵殑鍒ゆ柇

1. 鏈鐩栧弬鏁版椂 `count` 澶氬锛?2. `#(.Width(16))` 瀹炰緥鍖栧悗锛宍count` 澶氬锛?3. 鍙傛暟鑳藉惁鍦ㄨ姱鐗囪繍琛屼腑鍔ㄦ€佹敼鎴?16锛?
<details>
<summary>鍙傝€冨垎鏋?/summary>

榛樿 `Width=4`锛屾墍浠?`count` 鏄?4 浣嶃€傚疄渚嬪寲鏃惰鐩栦负 16 鍚庯紝瀵瑰簲瀹炰緥鐢熸垚 16 浣嶈鏁板櫒銆傚弬鏁板湪 elaboration/鐢熸垚纭欢鏃剁‘瀹氾紝涓嶆槸杩愯鏃朵俊鍙凤紱杩愯涓敼鍙樺搴︽槸涓嶅彲鑳界殑銆?
</details>

---

## 08锝渀parameter` 涓?`localparam`

### 浠ｇ爜鐗囨

```systemverilog
parameter int Width = 8;
localparam int CountWidth = $clog2(Depth + 1);
```

### 浣犵殑鍒ゆ柇

1. 涓よ€呰皝鍙互鐢辨ā鍧楀疄渚嬪寲鑰呰鐩栵紵
2. `CountWidth` 涓轰粈涔堜笉鏄洿鎺ュ啓姝讳负 3锛?
<details>
<summary>鍙傝€冨垎鏋?/summary>

`parameter` 鍙湪瀹炰緥鍖栨椂瑕嗙洊锛沗localparam` 鍙兘鍦ㄦā鍧楀唴閮ㄤ娇鐢紝涓嶈兘琚閮ㄦ敼鍐欍€俙CountWidth` 鐢?`Depth` 鎺ㄥ锛屾繁搴﹀彉浜嗚鏁板櫒瀹藉害涔熶細鑷姩鏇存柊锛涜繖灏辨槸鍙傛暟鍖栨ā鍧楅伩鍏嶁€滄敼浜嗕竴涓暟瀛椼€佸繕浜嗘敼鍙﹀涓変釜鏁板瓧鈥濈殑鏂规硶銆?
</details>

---

## 09锝減acked 涓?unpacked 鏁扮粍

### 浠ｇ爜鐗囨

```systemverilog
logic [7:0] byte_data;
logic [7:0] mem [0:3];
```

### 浣犵殑鍒ゆ柇

1. `byte_data` 鏈夊灏戜綅锛?2. `mem` 鎬诲叡鑳藉瓨澶氬皯涓瓧鑺傦紵
3. `mem[2]` 鐨勪綅瀹芥槸浠€涔堬紵

<details>
<summary>鍙傝€冨垎鏋?/summary>

`byte_data` 鏄竴涓?packed 鐨?8 浣嶅悜閲忋€俙mem` 鏄?4 涓厓绱犵粍鎴愮殑 unpacked 鏁扮粍锛屾瘡涓厓绱犻兘鏄?8 浣嶏紝鍥犳鎬诲閲忎负 4 瀛楄妭銆俙mem[2]` 閫夊嚭绗?3 涓厓绱狅紝浠嶇劧鏄?8 浣嶃€侳IFO 甯哥敤杩欑鍐欐硶鎻忚堪瀛樺偍鍣ㄣ€?
</details>

---

## 10锝滃瓨鍌ㄥ櫒鍐欏叆

### 浠ｇ爜鐗囨

```systemverilog
if (wr_en && !full) begin
    mem[wr_ptr] <= wr_data;
    wr_ptr      <= wr_ptr + 1'b1;
end
```

### 浣犵殑鍒ゆ柇

1. 鍝竴涓湴鍧€琚啓鍏ワ紵
2. 鍐欏叆鍜屾寚閽堝姞涓€鐨勫厛鍚庡叧绯绘槸浠€涔堬紵
3. `full=1` 鏃朵細鍙戠敓浠€涔堬紵

<details>
<summary>鍙傝€冨垎鏋?/summary>

鍐欏叆浣跨敤**鏃?* `wr_ptr` 鎸囧悜鐨勪綅缃紱鐢变簬閮芥槸闈為樆濉炶祴鍊硷紝鍐欏叆鍜屾寚閽堟洿鏂伴兘鍦ㄨ鏃堕挓娌垮悗鎻愪氦锛屼簩鑰呬笉浼氫簰鐩告姠鍏堛€俙full=1` 鏃舵暣涓潯浠朵负鍋囷紝鏃笉瑕嗙洊瀛樺偍鍣紝涔熶笉鎺ㄨ繘鍐欐寚閽堬紝杩欏氨鏄啓渚х殑娴佹帶銆?
</details>

---

## 11锝淔IFO 鐨勭粍鍚堣

### 浠ｇ爜鐗囨

```systemverilog
assign rd_data = empty ? 8'h00 : mem[rd_ptr];
```

### 浣犵殑鍒ゆ柇

1. `empty=1` 鏃朵负浠€涔堜笉鐩存帴璇?`mem[rd_ptr]`锛?2. `rd_data` 鍦ㄨ鎸囬拡鍙樺寲鍚庝綍鏃跺彉鍖栵紵
3. 杩欐槸鍚屾璇?RAM 杩樻槸缁勫悎璇?RAM 妯″瀷锛?
<details>
<summary>鍙傝€冨垎鏋?/summary>

绌?FIFO 娌℃湁鏈夋晥闃熷ご鏁版嵁锛屽洜姝ょ粰涓€涓‘瀹氬€?0锛岄伩鍏嶆尝褰腑璇敤鏃у瓨鍌ㄥ唴瀹广€俙rd_ptr` 鎴栫浉搴斿瓨鍌ㄥ崟鍏冨彉鍖栧悗锛宍rd_data` 缁忕粍鍚堥€昏緫鏇存柊銆傝繖鏄粍鍚堣妯″瀷锛涙煇浜?FPGA 鐨勫潡 RAM 鏄悓姝ヨ锛屾帴鍙ｄ笌鏃跺簭浼氫笉鍚岋紝鍚庣画闇€瑕佷笓闂ㄩ€傞厤銆?
</details>

---

## 12锝滅┖涓庢弧鐢辫鏁板櫒鎺ㄥ

### 浠ｇ爜鐗囨

```systemverilog
logic [2:0] count;
assign empty = (count == 0);
assign full  = (count == 4);
```

### 浣犵殑鍒ゆ柇

1. 娣卞害涓?4 鐨?FIFO锛宍count` 涓轰粈涔堣 3 浣嶏紵
2. `count=3` 鏃剁┖鍜屾弧鍒嗗埆鏄粈涔堬紵
3. 濡傛灉鎶?`count` 鍙啓鎴?2 浣嶏紝浠€涔堝€兼棤娉曡〃绀猴紵

<details>
<summary>鍙傝€冨垎鏋?/summary>

娣卞害 4 闇€瑕佽〃绀?0銆?銆?銆?銆? 浜斾釜鏁伴噺锛屽洜姝よ嚦灏?3 浣嶃€俙count=3` 鏃舵棦涓嶇┖涔熶笉婊°€? 浣嶅彧鑳借〃绀?0 鍒?3锛屾棤娉曡〃绀衡€滃凡瑁呮弧鐨?4 涓厓绱犫€濄€?
</details>

---

## 13锝滆鍐欐潯浠舵嫾鎺?
### 浠ｇ爜鐗囨

```systemverilog
case ({(wr_en && !full), (rd_en && !empty)})
    2'b10: count <= count + 1'b1;
    2'b01: count <= count - 1'b1;
    default: ;
endcase
```

### 浣犵殑鍒ゆ柇

1. 鎷兼帴琛ㄨ揪寮忕殑楂樹綅鍜屼綆浣嶅垎鍒槸浠€涔堬紵
2. `2'b11` 涓轰粈涔堣惤鍦?`default`锛?3. 鍚屾椂璇诲啓鏃?count 搴旇鍙樺寲鍚楋紵

<details>
<summary>鍙傝€冨垎鏋?/summary>

楂樹綅鏄€滄垚鍔熷啓鍏モ€濓紝浣庝綅鏄€滄垚鍔熻鍙栤€濄€?0 琛ㄧず浠呭啓鍏ワ紝鍏冪礌鏁板姞涓€锛?1 琛ㄧず浠呰鍙栵紝鍏冪礌鏁板噺涓€銆?1 琛ㄧず鍚屾椂鎴愬姛璇诲啓锛屼竴涓繘銆佷竴涓嚭锛屽厓绱犳暟閲忎笉鍙橈紝鎵€浠ュ彲浠ョ敱 `default` 淇濇寔銆傝繖閲?`default: ;` 琛ㄧず涓嶆墽琛岄澶栬鍙ャ€?
</details>

---

## 14锝滃悓鍛ㄦ湡璇诲啓鐨勯澶栭棶棰?
### 浠ｇ爜鐗囨

```systemverilog
2'b11: begin
    mem[wr_ptr] <= wr_data;
    wr_ptr      <= wr_ptr + 1'b1;
    rd_ptr      <= rd_ptr + 1'b1;
end
```

### 浣犵殑鍒ゆ柇

1. 涓轰粈涔?count 淇濇寔涓嶅彉锛?2. 褰?`wr_ptr == rd_ptr` 鏃讹紝璇诲嚭鐨勫埌搴曟槸鏃ф暟鎹繕鏄柊鏁版嵁锛?3. 涓轰粈涔堣繖鏄?FIFO 璁捐閲岄渶瑕佹槑纭鏍肩殑涓€鐐癸紵

<details>
<summary>鍙傝€冨垎鏋?/summary>

涓€鍐欎竴璇伙紝鍏冪礌鍑€鏁伴噺涓?0锛屾墍浠?count 淇濇寔銆傝嫢璇诲啓鍚屼竴鍦板潃锛岀粨鏋滃彇鍐充簬 RAM 鐨?read-first/write-first/no-change 琛屼负浠ュ強浣犻噰鐢ㄧ殑璇绘帴鍙ｏ紱涓嶅悓 FPGA/ASIC 瀹忓彲鑳戒笉鍚屻€傚洜姝ょ湡瀹?FIFO 瑙勬牸蹇呴』鏄庣‘鍚屽懆鏈熻鍐欍€佺┖/婊¤竟鐣屾椂鐨勮涓猴紝涓嶈兘鍙潬鈥滅湅璧锋潵鍚堢悊鈥濄€?
</details>

---

## 15锝滆鍐欐寚閽堜负浠€涔堜細鍥炵粫

### 浠ｇ爜鐗囨

```systemverilog
logic [1:0] wr_ptr;

wr_ptr <= wr_ptr + 1'b1;
```

### 浣犵殑鍒ゆ柇

1. `wr_ptr=2'b11` 鍚庡姞涓€浼氬彉鎴愪粈涔堬紵
2. 杩欎负浠€涔堟濂介€傚悎娣卞害涓?4 鐨勫瓨鍌ㄥ櫒锛?3. 浠呴潬涓や釜 2 浣嶆寚閽堣兘鍚﹀尯鍒嗏€滅┖鈥濆拰鈥滄弧鈥濓紵

<details>
<summary>鍙傝€冨垎鏋?/summary>

2 浣?11 鍔犱竴鍚庤嚜鐒舵埅鏂负 00锛屽嵆鍥炵粫銆傛繁搴︿负 4 鐨勫瓨鍌ㄥ櫒鍦板潃鎭板ソ闇€瑕?2 浣嶏紝鍥犳杩欏緢鏂逛究銆備絾浠呮瘮杈冭鍐欐寚閽堜細閬囧埌鈥滅浉绛夋棦鍙兘鏄┖锛屼篃鍙兘缁曚簡涓€鍦堝悗婊♀€濈殑姝т箟锛涙湰棰橀€氳繃鐙珛 `count` 瑙ｅ喅锛屽彟涓€绉嶅父瑙佹柟妗堟槸鎵╁睍鎸囬拡棰濆姣旇緝鍥炵粫浣嶃€?
</details>

---

## 16锝滀粠 FIFO 鍥炲埌 AI 鑺墖

### 浠ｇ爜鐗囨

```text
DDR / 杈撳叆娴?鈫?FIFO / SRAM 鈫?MAC 闃靛垪 鈫?绱姞鍣?鈫?杈撳嚭 FIFO
```

### 浣犵殑鍒ゆ柇

1. FIFO 鍦?MAC 闃靛垪鍓嶉潰瑙ｅ喅浠€涔堥棶棰橈紵
2. `full` 搴旇鍙嶉缁欎笂娓革紝杩樻槸涓嬫父锛?3. `empty` 搴旇闃绘璋佺户缁彇鏁版嵁锛?
<details>
<summary>鍙傝€冨垎鏋?/summary>

FIFO 瑙ｈ€︾敓浜ц€呭拰娑堣垂鑰呯殑閫熺巼锛氫笂娓哥獊鍙戞彁渚涙暟鎹€丮AC 闃靛垪鏆傛椂鏉ヤ笉鍙婂鐞嗘椂锛孎IFO 缂撳啿鏁版嵁銆俙full` 搴斿弽棣堢粰涓婃父锛岃姹傚畠鍋滄鍐欏叆锛沗empty` 鍛婅瘔涓嬫父褰撳墠娌℃湁鍙鏁版嵁锛屽簲闃绘 MAC 鎴栬绔户缁彇鏁般€傚悗缁綘浼氭妸瀹冩墿灞曚负鐗囦笂 Buffer/Scratchpad銆?
</details>

---

## 瀹屾垚鏈壒鍚庣殑鑷祴

涓嶇湅绛旀锛岃В閲婁互涓嬩笁浠朵簨锛?
1. 涓轰粈涔堢粍鍚堥€昏緫蹇呴』瑕嗙洊鎵€鏈夎緭鍑鸿祴鍊艰矾寰勶紵
2. 娣卞害 4 鐨?FIFO 涓轰粈涔堥渶瑕?2 浣嶆寚閽堝拰 3 浣?count锛?3. 涓轰粈涔?AI 鍔犻€熷櫒涓嶈兘璁?MAC 闃靛垪鐩存帴闅忔剰璇诲彇澶栭儴鏁版嵁锛岃€岄渶瑕?Buffer/FIFO锛?
鑳借娓呭悗锛屼笅涓€鎵瑰皢杩涘叆锛歚valid/ready` 鎻℃墜銆佹祦姘村瘎瀛樺櫒銆佺姸鎬佹満鍜岀畝鍖栧瘎瀛樺櫒鎺ュ彛銆?

