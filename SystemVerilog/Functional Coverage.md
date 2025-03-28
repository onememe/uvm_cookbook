
# A Practical Look @ SystemVerilog Coverage

## Tips, Tricks, Gottchas

### Tip #1: Take advantage of shorthand notation

Systemverilog определяет много кратких путей определения покрытия, которого вы ищете. На примере - конечный автомат и мы собираемся определить покрытие перехода - запись перехода из одного состояния в другое. Заметьте, чтобы определить покрытие перехода, используется синтаксис (=>).

```systemverilog
enum {Idle, Standby, Go1, Go2} states;
covergroup cg_FSM @(posedge Clock);
	coverpoint State {
		bins valid_states[] = {Idle, Standby, Go1, Go2};
		bins valid_trans = (Idle => Go1 => Go2 => Idle),
						   (Idle, => Standby => Idle);

		// Shorthand notation ...
		bins reset_trans = (Go1, Go2, Standby => Idle);
		bins idle_5 = (Idle[*5] => Go1);  // 5 Idles then Go1
		bins go1_range = (Go1 [-> 5:7]);  // 5 to 7 non-consecutively
		wildcard bins idle_trans = (2'bx1 => Idle);
	}
endgroup
```

Некоторые сокращения включают:
1. `s1, s2, s3 => n1, n2` интерпретируется как `s1 => n1, s1 => n2, s2 => n1, ..., s3 => n2`
2. `[*n]` - оператор повтора
3. `[->n:m]` - непоследовательный оператор (т.е. )

### Tip #2: Add covergroup arguments for more flexibility

Группы покрытие также могут включать аргументы (используется тот же синтаксис, как в функциях или тасках). В примере ниже, был добавлен аргумент для `v` так, что внутрь группы покрытия можно передать сигнал или переменную, которую мы хотим покрыть. Следует отметить, что мы передаем аргумент по ссылке используя ключевое слово `ref`. Также можно передать другие аргументы, такие как строки, которые можно использовать в настройках группы покрытия.

```systemverilog
covergroup cg(ref int v, input string comment);
	coverpoint v;

	option.per_instance = 1;
	option.weight = 5;
	option.goal = 90;
	option.comment = comment;
endgroup

int a, b;

cg cg_inst1 = new(a, "This is cg_inst1 - variable a");
cg cg_inst2 = new(b, "This is cg_inst2 - variable b");
```

При добавлении аргументов мы можем создавать несколько экземпляров и передавать им сигналы или переменные, которые мы хотим покрыть, в вызов метода `new`. Это позволяет переиспользовать определения групп покрытия.

Группы покрытия могут содержать точки покрытия к иерархическим ссылкам, что может быть довольно полезно. Однако, они не могут ссылаться на другие точки покрытия, как иллюстрирует пример ниже

```systemverilog
covergroup cg;
	coverpoint testbench.covunit.a;
	coverpoint $root.test.count;
	coverpoint testbanch.covunit.cg_inst.cp_a;  // X (coverpoint refs not allowed)
endgroup
```

К сожалению, когда мы начинаем использовать захардкоженные иерархические ссылки, наша группа покрытия (и соответственно наше тестовое окружение) перестает быть гибким и переиспользуемым. Вместо этого, мы можем определить аргменты для нашей группы покрытия, а затем передать иерархические интерфейсы в группу покрытия, когда они создаются. Создание экземпляра может быть совершено в тестовом сценарии либо где-то еще, так что теперь группа покрытия более гибкая.

```systemverilog
covergroup cg(ref logic[7:0] a, ref int b);
	coverpoint a;
	coverpoint b;
endgroup
cg cg_inst = new(testbench.covunit.a, $root.test.count);
```


### Tip #3: Utilize coverage options

Группы покрытия имеют множество настроек, которые могут быть определены. Опции типа применяются ко всей группе покрытия и могут быть установлены, когда группа покрытия объявляется, либо при помощи оператора (::). Параметры типа определяются с использованием элемента группы покрытия `type_option`. Существует 4 параметра типа - `weight`, `goal`, `strobe`, `where`.

```systemverilog
covergroup cg @(posedge clk);
	type_option.weight = 5;  // weight in calcucation
	type_option.goal = 90;   // percentage of coverage
	type_option.strobe = 1;  // sample in postponed region
	cp_a: coverpoint a {
		type_option.comment = comment;
	};
	coverpoint b;
endgroup

cg::type_option.goal = 100;
cg::cp_a::type_option.weight = 80;
```

`weight` - весомость покрытия при расчете
`goal` - процент достижения покрытия 
`strobe` - сбор значений покрытия, когда все значения стабильны, прямо перед переходом к следующем шагу симуляции (т.е. к отложенной области моделирования)
`comment` - строковый комментарий

В общем, покрытие кумулятивно, пока не определить параметр `per_instance`. Когда он установлен, покрытие собирается раздельно для каждого экземпляра группы покрытия. Существует множество параметров `per_instance`, как показано ниже.

```systemverilog
covergroup cg @(posedge clk);
	option.per_instance = 1;
	option.weight = 5;
	option.goal = 90;
	option.at_least = 10;
	option.comment = comment;

	a: coverpoint a { option.auto_bin_max = 128; };
	b: coverpoint b { option.weight = 50; };

endgroup

cg g1 = new;
g1.option.goal = 100;
g1.a.option.weight = 80;
```

Следует обратить внимание на параметр `at_least`. Он определяет число случаев, которые должны быть покрыты, и используется для определения того, была ли достигнута цель группы покрытия. Заметьте, `per_instance` параметры определяются при помощи элемента группы покрытия `option` и в большинстве могут быть использованы с группами покрытия, точками покрытия, а также их пересечениями.

![[Pasted image 20250320131204.png]]

## 3. COVERAGE TRICKS
### 3.1 *Trick #1*: Combine cover properties with covergroups

Cover properties и covergroups существенно по разному используются. Cover property отлавливает совпадения временной последовательности либо события, в то время как группа покрытия создает ячейки различных значений по мере того как они встречаются. Однако, иногда полезно использовать мощные синтаксис временных свойств SVA, чтобы наблюдать за поведением и пересечением того покрытия с другими происходящими значениями или событиями. Например, свойство покрытия может с легкостью описывать транзакцию чтения или записи через интерфейс шины и будет полезно скрестить все транзакции чтения и записи и различные адресные пространства.

Свойства покрытия, мониторящие за транзакциями чтения и записи APB на системном интерфейсе, может выглядеть следующим образом:

### 3.2 *Trick #2*: Create coverpoints to query bin coverage

Группы и точки покрытия, а также их пересечения, имеют встроенную функцию `get_coverage()`, которая возвращает значение текущего процента покрытия. 

```systemverilog
initial 
	repeat (100) @(posedge clk) begin
		cg_inst.sample();  // sample coverage

		if (cg_inst.get_coverage() > 90.0)
			cg_inst.stop();
	end
```

Важно знать, как оно вычисляется - точка покрытия может быть покрыта, если все ее ячейки покрытия покрыты на 100%, иначе она считается непокрытой.

У бинов нет метода `get_coverage()`, но если он нужен, можно заворачивать бины в группы покрытия

```systemverilog
covergroup cg;
	coverpoint i {
		bins zero = {0};
		bins tiny = {[1:100]};
	}
endgroup

// ILLEGAL - not allowed!
// cov = cg_inst.i.zero.get_coverage();
```


### 3.3 *Trick #3*: Direct stimulus with coverage

С помощью встроенных в точки покрытия методов можно управлять генерацией стимулов и рандомизацией. Можно использовать, например, в `randcase`

```systemverilog
// bias randomness to hit uncovered coverpoints randcase
randcase 
	(101 - cg_inst.a.getcoverage): ...;
	(101 - cg_inst.b.getcoverage): ...;
	(101 - cg_inst.c.getcoverage): ...;
	...
endcase
```

Другой способ - использовать смещение распределения SystemVerilog `dist`. Однако, не все симуляторы это поддерживают

```systemverilog
int weight_nop = 1,
	weight_load = 1,
	weight_store = 1,
	weight_add = 1,
	...;

constraint bias_opcodes {
	opcode dist {
		nop_op := weight_nop,
		load_op := weight_load,
		store_op := weight_store,
		add_op := weight_add,
		...
	};
}

function real calc_weight(opcode_t op);
	real cov;
	case (op)
		nop_op:
			cov = covunit.cg.op_nop.get_coverage;
		...
	endcase
	calc_weight = (100 - cov) * 0.5;
endfunction

function void pre_randomize();
	...
```

Предостережение - при применении данных подходов рандомизация перестает быть рандомизацией по настоящему.


### 3.4 *Trick #4*: Covering cover properties

Когда свойства покрытия соответствует поведению, симулятор продолжает отслеживать количество попыток или совпадений. В отличие от групп покрытия, которые могут быть запрошены на покрытия, нет прямого способа доступа к информации об этом покрытии внутри SystemVerilog, вместо этого, он включен в отчет о покрытии симулятора. Аналогично не существует языковых конструкций, позволяющей взвешивать свойство покрытия в отчете о покрытии. Пока нет прямого способа для доступа к этой информации, но есть несколько окольных путей получить её.

#### *3.4.1 Solution 1: SystemVerilog only approach*

Самый простой способ выяснить число совпадений из свойства покрытия - просто отслеживать с помощью счетчика. Свойства покрытия позволяют выполнять выражения, когда происходит совпадение, в этот момент может инкрементироваться счетчик. Например, покрытия может сохраняться в ассоциативный массив:

```systemverilog
int coverage[string];  // coverage array

c1: cover property (a |=> b) coverage["c1"]++;
c2: cover property (c |=> d) coverage["c2"]++;
```

Это покрытие записывает число успешных выполнений последовательности, однако, можно сохранять также попытки исполнения последовательности, используя функцию:

```systemverilog
function void cov(string s);
	coverage[s]++;
endfunction

c1: cover property (a |=> (b, cov("c1")));
c2: cover property ((c, cov("c2")) |=> d);
```


