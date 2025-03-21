
# UVM Basics

## UVM Basics

Эта глава вводит в концепции, которые должен знать читатель для того, чтобы понять содержимое `cookbook`.

## Testbench Basics

UVM использует многоуровневый объекто-ориентированный подход для разработки тестовых окружений. UVM позволяет получать в результате модульные, переиспользуемые верификационные окружения, что позволяет тестировщику думать на уровне транзакций и фокусироваться на верификации.

Тестируемое устройство (ТУ/Design Under Test, DUT) - соединяется со слоем транзакторов, которые коммуницируют с ним на уровне выходов, а с остальным тестовым окружением UVM с помощью объектов транзакций. Слой тестового окружения над транзакторным состоит из компонентов, взаимодействующих на уровне транзакций. 

![[Pasted image 20241212184349.png]]

Низший уровень тестового окружения интерфейсо-зависим. Для каждого интерфейса UVM предоставляет агента (agent), который содержит драйвер (driver), монитор (monitor), секвенсор (sequencer) и, опционально, сборщик покрытия (coverage collector). Агент воплощает в себе специфичные для протокола коммуникации с ТУ. Агент и другие специфичные для устройства компоненты инкапсулированы в `uvm_env` компоненте окружения (environment), которое, в свою очередь, создается и настраивается `uvm_test` компонентной верхнего уровня.

`uvm_sequence_item` (элемент последовательности) - иногда упоминается как транзакция - это uvm_object, `содержащий` поля данных, необходимые для реализации протокола и коммуникации с тестируемым устройством. `uvm_driver` преобразовывает `sequence_item` в сигналы на выводах для того, чтобы отправить или прочесть информацию от ТУ. `sequence_item` поставляются одним или более объектами `uvm_sequence`, они определяют стимулы на уровне транзакций и исполняются с помощью `uvm_sequencer` компонента агента. Сиквенсор отвечает за выполнение последовательностей, разрешение споров между ними и проведение элементов последовательности между драйвером и последовательностью (sequence).

`uvm_monitor` пассивно наблюдает за поведением сигнального уровня интерфейса ТУ, преобразовывает сигналы в элементы последовательности и предоставляет эти элементы анализирующим компонентам в агенте или где-либо еще в тестовом окружении (сборщики покрытия или скорборды). В агентах также есть объект конфигурации, позволяющий тестировщику управлять аспектами агента при сборке и выполнении тестового окружения.

Предоставляя единый интерфейс тестового окружения, агент изолирует его и последовательности UVM от деталей реализации интерфейса. Так, последовательность, предоставляющая пакеты данных, может быть переиспользована разными агентами UVM, реализующими различные протоколы. В окружении UVM обычно для каждого интерфейса тестового устройства существует свой агент.

Для данного устройства компоненты инкапсулируются в компонент окружения `uvm_env`, который обычно специфичен для устройства. Как и агент, окружение обычно содержит свой объект конфигурации, позволяющий тесту управлять аспектами окружения так же, как и управлять агентом внутри этого окружения. Окружения могут быть собраны внутри другого окружения верхнего уровня. Как блочные устройства могут быть собраны в подсистемы и системы, блочные окружения, соответствующие блоку, могут быть переиспользованы как компоненты в подсистемном уровне окружения, которое в свою очередь может быть переиспользовано как тестовое окружение системного уровня.

Как только окружение/среда будет определена, `uvm_test` создаст его экземпляр, сконфигурирует и соберет, включая кастомизированные ключевые аспекты тестового окружения в целом (вариации используемых в окружении компонент, запускаемых последовательностей, конфигурационных объектов окружения, подокружений и агентов в тестовом окружении).

UVM тест начинается с `initial` блока на верхнем уровне HVL модуля вызовом  `run_test()`.


## UVM Components

Тестовое окружение UVM состоит из компонентов - объектов, наследующихся от базового класса `uvm_component`. Когда создается объект класса, наследуемого от `uvm_component`, он становится частью иерархии тестового окружения, которая существует на протяжении всего времени симуляции. Это контрастирует с веткой последовательности в классовой иерархии UVM, которая включает в себя недолговечные объекты - объекты, которые создаются, используются и уничтожаются (сборщиком мусора) после удаления ссылки на них.

![[Pasted image 20241213132504.png]]

(Квази) статичная иерархия компонентов в UVM используется инфраструктурой отчётности, чтобы вывести область видимости компоненты, выводящей сообщение, также конфигурационным процессом, чтобы определить какие компоненты могут иметь доступ к конфигурационном объекту, а также фабрикой UVM, чтобы применять переопределения. Эта иерархия компонентов представляется связанным списком, постепенно увеличивающимся по мере создания каждого компонента. Иерархическое положение каждой компоненты определено переданными аргументами: именем и родителем, переданными в метод создания во время конструирования. 

Например, фрагмент кода ниже - компонента `apb_agent`, создается внутри `spi_env`. Предполагая, что `spi_env` создается в компоненте теста верхнего уровня и имеет имя `m_env`, иерархический путь агента - конкатенация имени компоненты окружения `uvm_test_top.m_env`, оператора точки `.` и имени, переданного первым аргументом в метод `create()`, результирующее иерархическое имя для агента - `uvm_test_top.m_env.m_apb_agent`. Любые ссылки к агенту должны использовать это строковое имя.

```systemverilog
/* Hierarchical name example */
class spi_env extends uvm_env;
	// ...
	apb_agent m_apb_agent;  // declaration of the apb agent handle
	// ...
	function void build_phase(uvm_phase phase);
		m_apb_agent = apg_agent::type_id::create("m_apb_agent", this);
	endfunction: build_phase
	// ...
endclass: spi_env
```

Класс `uvm_component` наследуется от класса `uvm_report_object`, который лежит в самом сердце инфраструктуры сообщений UVM.  Процесс отчетности использует статическую иерархию, чтобы добавить область расположения компоненты в строку сообщения.

Шаблон базового класса `uvm_component`содержит виртуальный метод для каждой фазы UVM и они должны быть реализованы пользователем по мере необходимости. Если виртуальный метод какой-либо фазы не реализован, компонента эффективно не принимает участия в этой фазе.

Базовый класс также поддерживает таблицу конфигурации. При использовании API `uvm_config_db` статическая иерархия используется как часть механизма нахождения пути, чтобы управлять доступом компонентов к конфигурационному объекту.

Для гибкости конфигурирования и более умного построения конфигурации, компоненты UVM регистрируются в фабрике UVM (UVM factory). Фабрика позволяет менять компоненты на другие, совместимые производные типы используя переопределение. Это удобная техника для изменения функциональности тестового окружения без изменений в коде напрямую, что потребует перекомпиляции и будет препятствовать переиспользованию.

Пакет UVM содержит несколько расширений класса `uvm_component` для общепринятых компонентов тестового окружения. Большинство из них очень "стройны", являются буквально небольшими расширениями базового класса. Хотя все еще можно наследоваться от `uvm_component`, встроенные расширения "документируют" код. Вдобавок, в них есть утилиты для анализа, которые также используют эти посторонние классы как ключи, чтобы установить картину тестовой иерархии. С другой стороны, некоторые встроенные расширения `uvm_component` являются строительными блоками, имеющими более глубокое значение за счет создания конкретных под-компонентов.

![[Pasted image 20241213142236.png]]

## The UVM Factory

### The UVM Factory

Предназначение фабрики - возможность объект одного типа заместить объектом  унаследованного типа без изменений структуры тестового окружения или даже его кода. Механизм называется переопределением по типу либо экземпляру. Заменяемые компоненты должны быть полиморфически совместимы. Это включает в себя необходимость создания заменяемым компонентом тех же самых TLM интерфейсов и объектов. В добавок, для работы фабрики должны соблюдаться некоторые соглашения о кодировании.

### Factory Coding Convention 1: Registration

Компонента или объект должны содержать код регистрации в фабрике, состоящий из следующих элементов:

- Обертку `uvm_component_registry`, тип которой определен как `type_id`
- Статическую функцию получения `type_id`
- Функцию получения имени типа

Пример:

```systemverilog
class my_component extends uvm_component;
	// wrapper class around the component class that is
	// used within the factory
	typedef uvm_component_registry #(my_component, "my_component") type_id;

	// used to get the type_id wrapper
	static function type_id get_type();
		return type_id::get();
	endfunction

	// used to get the type_name as a string
	function string get_type_name();
		return "my_component";
	endfunction

	// ...
endclass: my_component
```

У кода регистрации есть стандартный шаблон, который может быть сгенерирован одним из четырех макросов регистрации в фабрике:

```systemverilog
// for a component
class my_component extends uvm_component;
	// component factory registration macro
	`uvm_component_utils(my_component)
	...

// for a parameterized component
class my_param_component #(int ADD_WIDTH=20, int DATA_WIDTH=23)
	extends uvm_component;

	typedef my_param_component #(ADD_WIDTH, DATA_WIDTH) this_t;
	// parameterized component factory registration macro
	`uvm_component_param_utils(this_t)
	...

// for a class derived from an object (i.e. uvm_object,
// uvm_transaction, uvm_sequence_item, uvm_sequence etc.)
class my_item extends uvm_sequence_item;
	`uvm_object_utils(my_item)
	...

// for a parameterized object class
class my_item #(int ADD_WIDTH=20, int DATA_WIDTH=20)
	extends uvm_sequence_item;

	typedef my_item #(ADD_WIDTH, DATA_WIDTH) this_t;
	`uvm_object_param_utils(this_t)
	...
```

### Factory Coding Convention 2: Constructor Defaults

Конструкторы компонент и объектов - виртуальные методы с прототипом шаблона, которого должны придерживаться пользователи. Для того, чтобы поддерживать отложенное конструирование во время `build` фазы, конструктор фабрики должен содержать значения по умолчанию для аргументов конструктора. Это позволяет зарегистрированном в фабрике классу быть включеным в фабрику с использованием значений по умолчанию и тогда свойства класса могут быть переопределены аргументами, переданными через метод `create` класса оберкти `uvm_component_registry`. Значения по умолчанию различны для компонентов и для объектов:

```systemverilog
// for a component:
class my_component extends uvm_component;
	function new(string name = "my_component", uvm_component parent = null);
		super.new(name, parent);
	endfunction

// for an object
class my_item extends uvm_sequence_item;
	function new(string name = "my_item");
		super.new(name);
	endfunction
```

### Factory Coding Convention 3: Component and Object Creation

Компоненты тестового окружения создаются во время `build` фазы, используя метод `create`, принадлежащий `uvm_component_registry`. Он сначала конструирует класс, затем присваивается дескриптор класса его дескриптору в тестовом окружении, после правильного указания аргументов имени и родителя. Для компонентов процесс сборки происходит сверху вниз, что позволяет компонентам более высокого уровня конфигурировать и управлять сборкой.

Объекты класса создаются по мере необходимости, используя тот же `create` метод. Следующий пример кода иллюстрирует как это сделано:

```systemverilog
class env extends uvm_env;
	my_component m_my_component;
	my_param_component #(.ADDR_WIDTH(32), .DATA_WIDTH(32))
		m_my_p_component;

	// constructor & registration left out
	// component and parameterized component create examples
	function void build_phase(uvm_phase phase);
		m_my_component = my_component::type_id::create(
			"m_my_component", this);
		m_my_p_component = my_param_component #(32, 32)::type_id::create(
			"m_my_p_component", this);
	endfunction: build

	task run_phase(uvm_phase phase);
		my_seq test_seq;
		my_param_seq #(.ADDR_WIDTH(32), .DATA_WIDTH(32)) p_test_seq;

		// object and parameterized object create examples
		test_seq = my_seq::type_id::create("test_seq");
		p_test_seq = my_param_seq #(32, 32)::type_id::create(
			"p_test_seq", this);
		// ...
	endtask: run_phase
```

## Phasing

### The Standard UVM Phases

Для того, чтобы иметь последовательный поток исполнения тестового окружения, UVM использует фазы, чтобы упорядочить главные шаги, происходящие во время симуляции. Существует три группы фаз, которые исполняются в следующем порядке:
1. Build фаза - в которой тестовое окружение настраивается и конструируется
2. Run-time фаза - продвигает время симуляции для исполнения тестовых сценариев на тестовом окружении
3. Clean up фаза - в ней результаты тестовых сценариев собираются в отчете
Разница между этими группами иллюстрируется на диаграмме снизу. Класс `uvm_component` содержит виртуальные методы, которые вызываются в соответствующем методе различных фаз, эти методы заполняются создателем компоненты в соответствии с тем, в каких фазах принимает участие компонент. Использование различных фаз позволяет верификационным компонентам разрабатываться по отдельности, но все равно быть совместимыми, поскольку существует общедоступное понимание, что должно происходить в той или иной фазе.

![[Pasted image 20241213185939.png]]

### Starting UVM Phase Execution

Для того, чтобы запустить тестовое окружение, из статической части тестового окружения должен быть вызван метод `run_test()`. Он обычно вызывается изнутри блока `initial` модуля верхнего уровня (`top`) тестового окружения.

Вызов этого метода конструирует коренной компонент (`root`) окружения UVM, а затем инициирует `phasing`. В метод может быть передан строковый аргумент, определяющий имя `uvm_component` класса, который должен использоваться как корень тестовой иерархии. Однако метод проверяет аргумент командной строки (`plusarg +UVM_TESTNAME`) и перезаписывает им имя типа по умолчанию. `root node` определяет выполняемый тестовый кейс, определяя конфигурацию компонент тестового окружения и исполняемые ими стимулы.

Тестовый компонент `my_test` определяется классом-корнем тестового окружения:
```bash
vsim tb_top +UVM_TESTNAME=my_test
```

...

## UVM Driver

Ответственен за коммуникацию на уровне транзакций `sequence`-ами через `TLM` с `sequencer`, а также за преобразование между `sequence_item` в транзакции на уровне сигналов - коммуникация с тестируемым устройством через виртуальный интерфейс. Может получать ответ от ТУ и посылать его обратно в `sequence_item` для завершения транзакции. Драйвер может также функционировать как "ответчик" в (режиме ведомого). Когда агент сконфигурирован в пассивном режиме, драйвер не создается.

### Anatomy of a UVM Driver

Определенный пользователем компонент `driver` содержит BFM (Bus Functional Model) - интерфейс SystemVerilog.

## UVM Monitor

Также осуществляет перевод между самой сигнальной активностью и абстрактным представлением этой активности. Всегда пассивен, не подает никаких сигналов на интерфейс.

### Construction

Монитор состоит из класса прокси (`uvm_monitor`) и BFM интерфейса SystemVerilog. Прокси должен иметь один порт анализа и дескриптор виртуального интерфейса, указывающий на BFM интерфейс.

```systemverilog
class wb_bus_monitor extends uvm_monitor;
	`uvm_component_utils(wb_bus_monitor)

	uvm_analysis_port #(wb_txn) wb_mon_ap;
	virtual wb_bus_monitor_bfm m_bfm;
	wb_config m_config;

	`uvm_component_new

	function void build_phase(uvm_phase phase);
		wb_mon_ap = new("wb_mon_ap", this);
		m_config = wb_config::get_config(this);
		m_bfm = m_config.WB_mon_bfm;
		m_bfm.proxy = this;
	endfunction

	task run_phase(uvm_phase phase);
		m_bfm.run();
	endtask

	function void notify_transaction(wb_txn item);
		wb_mon_ap.write();
	endfunction: notify_transaction

endclass

interface wb_bus_monitor_bfm(wishbone_bus_syscon_if wb_bus_if);

	import wishbone_pkg::*;
	wb_bus_monitor proxy;
	
	task run();
		wb_txn txn;
		forever @(posedge wb_bus_if.clk)
			proxy.notify_transaction(txn);
		end
	endtask
	
endinterface
```

### Recognizing Protocols

Монитор должен знать протокол, чтобы обнаружить узнаваемые шаблоны сигнальной активности. Обнаружение может быть совершено кодом в таске `run()` BFM монитора. Этот код ждет шаблонную активность наблюдая за пинами интерфейса. 

### Building Transaction Objects

Когда шаблон замечен, монитор строит одни или возможно больше транзакций, которые абстрактно представляют сигнальную активность.

### Copy-on-Write Policy

Когда монитор записывает транзакцию в порт анализа, только указатель копируется и передается подписчикам. Операция записи может происходить каждый раз как монитор проходит через цикл признания шаблона в таске `run()`. Для того чтобы предотвратить перезапись одного и того же объектам памяти, передаваться должен указатель на отдельную копию объекта транзакции, которую создает монитор.

Это может быть достигнуто двумя способами:
- Создание нового объекта в каждой итерации цикла
- Использование одного и того же объекта, но записывать его клон

### Example Monitor

```systemverilog
task run();
	wb_txn txn;
	forever @(posedge wb_bus_if.clk)
		if (wb_bus_if.s_cyc) begin
			`uvm_create_obj(wb_txn, txn)
			txn.adr = wb_bus_if.s_addr;
			txn.count = 1;
			if (wb_bus_if.s_we) begin
				txn.data[0] = wb_bus_if.s_wdata;
				txn.txn_type = WRITE;
				while (!(wb_bus_if.s_ack[0] |
					wb_bus_if.s_ack[1] } wb_bus_if.s_ack[2]))
					@(posedge wb_bus_if.clk);
			end
			else begin
				txn.txn_type = READ;
				case (1)
					wb_bus_if.s_stb[0]: begin
						while (!(wb_bus_if.s_ack[0]))
							@(posedge wb_bus_if.clk);
						txn.data[0] = wb_bus_if.s_rdata[0];
					end
					wb_bus_if.s_stb[1]: begin
						while (!(wb_bus_if.s_ack[1]))
							@(posedge wb_bus_if.clk);
						txn.data[0] = wb_bus_if.s_rdata[1];
					end
				endcase
			end
			proxy.notify_transaction(txn);
		end
endtask
```

## UVM Agent

UVM Agent - набор верификационных компонентов для данного логического интерфейса (например APB или USB). Агент содержит: SystemVerilog интерфейс, включающий в себя соответствующий набор сигналов интерфейса, два SystemVerilog интерфейса, представляющих BFM модели монитора и драйвера, а также SystemVerilog пакет, включающий различные классы, составляющие общий компонент класса агент. Агент сам по себе - контейнер классов `sequencer`, `driver` и `monitor` proxy плюс некоторые другие верификационные компоненты (`functional coverage collector` or `scroreboard`). `driver` и `monitor` прокси общаются с остальным тестовым окружением UVM, а также имеют доступ к BFM интерфейсам через через дескриптор виртуального интерфейса. Следовательно, завершенный монитор состоит из прокси монитора и BFM монитора, работающих вместе в паре... Агент также имеет порт анализа, соединенный с портом монитора, позволяющий пользователю подключать внешние компоненты анализа к агенту без необходимости знать внутреннюю структуру. Агент - блок низшего уровня иерархии в тестовом окружении, его структура зависит от конфигурации, которая может различаться от одного теста к другому.  Классы и интерфейсы вместе составляют портативный или переиспользуемый Агент.

![[Pasted image 20250114191849.png]]

Давайте исследуем, как APB агент сформирован, настроен, собран и соединен. `pin` интерфейс агента APB, `apb_if`, написан в файле `apb_if.sv`. Монитор BFM интерфейса с именем `apb_monitor_bfm` содержит порт `apb_if`. Драйвер BFM с названием `apb_driver_bfm` также содержит `apb_if` порт. Функциональные модели шины определяют задачи и функции (`task` и `functions`) для взаимодействия с сигналами на сигнальном интерфейсе `apb_if`. Представители драйвера и монитора не имеют прямого доступа к сигналам, которые должны храниться локально в функциональных моделях. Файл `apb_agent_pkg.sv` содержит пакет SystemVerilog с различными файлами классов для агента APB. Любой компонент, используемый файлы из этого пакета (например `env`), должен его импортировать.

```systemverilog
package apb_agent_pkg;
	import uvm_pkg::*;
	`include "uvm_macros.svh"
	`include "config_macro.svh"

	`include "apb_seq_item.svh"
	`include "apb_agent_config.svh"
	`include "apb_driver.svh"
	`include "apb_coverage_monitor.svh"
	`include "apb_monitor.svh"
	typedef uvm_sequencer #(apb_seq_item) apb_sequencer;
	`include "apb_agent.svh"

	// Reg adapter for UVM Register Model
	`include "reg2apb_adapter.svh"

	// Utility Sequences
	`include "apb_seq.svh"
	`include "apb_read_seq.svh"
	`include "apb_write_seq.svh"

endpackage: apb_agent_pkg
```

### The Agent Configuration Object

У агента есть объект конфигурации, который определяет:
- Топологию подкомпонентов агента (определяет что будет сконструировано)
- Дескрипторы для виртуальных интерфейсов BFM, используемые прокси драйвера и монитора
- Поведение агента

По соглашению, объект класса конфигурации агента содержит поле типа `uvm_active_passive_enum`, которое определяет активен агент или пассивен (`sequncer` и `driver`). Этот параметр называется `active` и по умолчанию установлен в `UVM_ACTIVE`.

Будут ли созданы другие подкомпоненты определяется дополнительными элементами конфигурации. Например, если есть сборщик функционального покрытия, должен быть бит, управляющий его созданием, с подходящим именем (`has_functional_coverage`).

Объект конфигурации хранит дескрипторы на виртуальные интерфейсы функциональных моделей шин. Конфигурационный объект создается и настраивается в тесте на верхнем уровне, где дескрипторы виртуальных интерфейсов назначаются интерфейсам и передаются внутрь из модуля тестового окружения.

Объект конфигурации может также содержать другие данные для управления настройкой поведением агента. Например, конфигурационный объект APB агента содержит члены для установки памяти и т. д.

Класс объекта конфигурации должен по умолчанию устанавливать свои настройки в частые значения.

Следующий пример кода показывает конфигурационный объект для агента APB.

```systemverilog
// Class Description:
class apb_agent_config extends uvm_object;
	`uvm_object_utils(apb_agent_config)

	virtual apb_monitor_bfm mon_bfm;
	virtual apb_driver_bfm drv_bfm;

	bit has_functional_coverage;
	bit has_scoreboard;

	int no_select_lines = 1;
	int apb_index = 0;
	logic[31:0] start_address[15:0];
	logic[31:0] range[15:0];

	extern function new(string name = "apb_agent_config");
endclass: apg_agent_config
```

### The Agent Build Phase


# Testbench Architecture

## Sequencer-Driver Connections | Connecting the Sequencer and Driver

Передача элементов последовательности запросов и ответов между последовательностями и их целевыми драйверами упрощается двунаправленным TLM механизмом коммуникации, реализуемой в сиквенсоре. `uvm_driver` содержит `uvm_seq_item_pull_port`, который должен быть соединен с `uvm_seq_item_pull_export` в соответствующем драйверу сиквенсоре. Когда они соединены, драйвер может использовать в своем коде API, чтобы получать sequence_items запросов от последовательностей и возвращать им ответы.

Связь между портом драйвера и экспортом сиквенсора производится с использованием метода соединения TLM во время `connect` фазы:

```systemverilog
// Driver parameterized with the same sequence_item for request & response
// defaults to request
class adpcm_driver extends uvm_driver #(adpcm_seq_item);
	// ...
endclass: adpcm_driver

// Agent containing a driver and a sequencer - uninteresting bits left
// out
class adpcm_agent extends uvm_agent;

	apdcm_driver m_driver;
	adpcm_agent_config m_cfg;
	// uvm_sequencer parameterized with the adpcm_seq_item
	uvm_sequencer #(adpcm_seq_item) m_sequencer;

	// sequencer-driver connection:
	function void connect_phase(uvm_phase phase);
		if (m_cfg.active == UVM_ACTIVE) begin
			m_driver.seq_item_port.connect(m_sequencer.seq_item_export);
			m_driver.vif = cfg.vif;
		end
	endfunction
```

Соединение между драйвером и сиквенсором обычно производится в `connect_phase()` методе агента. Связь - один к одному. В добавок к этому двунаправленному TLM порту есть в драйвере `analysis_port`, который может быть присоединен к `analysis_export` сиквенсора. Это исторический артефакт и он предоставляет избыточную функциональность, поэтому в основном не используется.


# Sequences

## Driver-Sequence API

`uvm_driver` - расширение класса `uvm_component`, с добавлением `uvm_seq_item_pull_port`, который используется для коммуникации с последовательностью через сиквенсор. `uvm_driver` - параметризированный типом элемента последовательности запроса и ответа класс. Эти параметры используются для параметризации `uvm_seq_item_pull_port`. Элемент последовательности ответа может отличаться от запроса. На практике, чаще всего драйверы используют одинаковые элементы последовательности как при получении запросов, так и для формирования ответов, поэтому, по умолчанию тип запроса присваивается типу ответа.

Модель использования класса `uvm_driver` - он потребляет элементы последовательностей запросов (REQ) из очереди в сиквенсоре, используя механизм рукопожатий, и, опционально, возвращает ответный элемент последовательности (RSP) сиквенсеру в другую очередь. Дескриптор `seq_item_pull_port` в `uvm_driver` - это имеет имя `seq_item_port`. API, используемое драйвером для взаимодействия с сиквенсером представляется `seq_item_port`, но, на самом деле, реализовано в `seq_item_export` сиквенсеров.

### UVM Driver API

API между драйвером и сиквенсором:

#### `get_next_item`

Метод блокируется, пока `sequence_item` запроса не станет доступным в очереди запросов сиквенсора, а затем вернет указатель на объект запроса

Вызов `get_next_item()` реализует половину протокола рукопожатия `driver-sequencer`, и за ним должен следовать вызов метода `item_done()`, завершающий рукопожатие. Попытка получения следующего запроса перед завершением предыдущего приведет к ошибке протокола и к остановке `driver-sequencer`.

#### `try_next_item`

Не-блокирующий вариант метода `get_next_item()`. Вернет null-pointer если нет доступного запроса элемента последовательности в очереди запросов сиквенсора. Однако, если запрос доступен, метод завершит первую часть рукопожатия драйвер-сиквенсор и должен иметь после себя вызов метода `item_done()`, для завершения рукопожатия.

#### `item_done`

Неблокирующий метод, завершающий рукопожатие. Должен быть вызван после успешного вызова `.._next_item()` метода.

Если в него не передается аргумента или null-pointer, от завершит рукопожатие не помещая ничего в очередь ответов сиквенсора. Если же в него передать указатель на ответный элемент последовательности в качестве аргумента, тогда указатель будет помещен в очередь сиквенсора.

#### `peek`

Если нет доступного запроса, метод `peek()` заблокирует поток до момента, когда запрос появится в очереди сиквенсора, а затем вернет указатель на объект запроса. При вызове выполняется первая половина рукопожатия драйвер-сиквенсор. Любые последующие вызовы метода `peek()` перед вызовом `get()` или `item_done()` вернут указатель на тот же запрос, что и в первый раз.

#### `get`

Блокирует поток исполнения, пока в очереди запросов сиквенсора не появится доступного элемента последовательности. Как только это произойдет, метод завершит протокол рукопожатия и вернет указатель на объект запроса.
#### `put`

Неблокирующий метод, используемый, чтобы разместить ответный элемент последовательности в очередь ответов сиквенсора. Метод `put()` может быть вызван в любое время и не имеет отношения к механизму рукопожатия драйвер-сиквенсор.



# The UVM Messaging System

...

## Using Messaging


### Message IDs

Поля ID сообщений UVM могут использоваться для группирования связанных сообщений и совместного управления ими. Строка ID может иметь произвольную форму.
### Message Actions

| `UVM_ACTION_e`  | Description                                               |
| --------------- | --------------------------------------------------------- |
| `UVM_NONE`      | Никаких действий - способ подавить сообщение              |
| `UVM_DISPLAY`   | Сообщение направляется в симуляцию                        |
| `UVM_LOG`       | Сообщение направляется в лог файл                         |
| `UVM_COUNT`     | Сообщение увеличивает значение счетчика выхода            |
| `UVM_EXIT`      | Сообщение тут же завершает симуляцию                      |
| `UVM_STOP`      | Сообщение ставит симуляцию на паузу                       |
| `UVM_RM_RECORD` | Сообщение логгируется в БД транзакций (`uvm_tr_database`) |
Эти действия кодируются в виде битового шаблона и могут быть объединены, чтобы создать определение гибридного действия.

Для любой компоненты, действия для различных типов компонент могут быть изменены в соответствии с их строгостью (severity) и/или id. API для изменения действий с сообщениями:

```systemverilog
// Apply to a single level of the component hierarchy:
set_report_severity_action(uvm_severity severity, uvm_action action);
set_report_id_action(string id, uvm_action action);
set_report_severity_id_action(uvm_severity severity, string id, uvm_action
							 action);

// For instance:
// Any `uvm_info() messages from this_component with an id of "this_agent"
// are sent to a log file and transcript
this_component.set_report_severity_id_action(UVM_INFO, "this_agent", 
											UVM_LOG | DISPLAY);

// Apply to all components below the component in the hierarchy:
set_report_severity_action_hier(uvm_severity severity, uvm_action action);
set_report_id_action_hier(string id, uvm_action action); set_report_severity_id_action_hier(uvm_severity severity, string id, 
								  uvm_action action);
```

**Using UVM_LOG**

Для того, чтобы использовать действие `UVM_LOG`, нужно взять на себя ответственность за открытие файла перед записью каких-либо сообщений, а также закрыть файл в конце симуляции. Вам также нужно передать дескриптор файла системе обмена сообщений. Лог файл также может быть настроен по умолчанию, или в соответствии со строгостью и/или id.
Вызываемые API:

```systemverilog
// Apply to a single leveel of the component hierarchy:
set_report_default_file(UVM_FILE file);
set_report_id_file(string id, UVM_FILE file);
set_report_severity_file(uvm_severity severity, UVM_FILE file);
set_report_severity_id_file(uvm_severity severity, string id, UVM_FILE file);

// Apply to all components below the component in the hierarchy:
set_report_default_file_hier(UVM_FILE file);
set_report_id_file_hier(string id, UVM_FILE file);
set_report_severity_file_hier(uvm_severity severity, UVM_FILE file); set_report_severity_id_file_hier(uvm_severity severity, string id, UVM_FILE file);
```

Следующий код - пример создания дескриптора лог файла, его назначения а затем закрытия:

```systemverilog
task run_phase(uvm_phase phase);
	// Create a file handle by opening a file:
	UVM_FILE green_log_fh = $fopen("green_messages.log");

	env.green.set_report_id_action("green_id", (UVM_DISPLAY | UVM_LOG));
	env.greet.set_report_if_file("green_id", green_log_fh);

	phase.raise_objection(this);
	#1us phase.drop_objection(this);

	$fclose(green_log_fh);
endtask
```

**Using UVM_COUNT**

`UVM_COUNT` полезно для управления тем, сколько ошибок может случиться перед тем как тест прервется