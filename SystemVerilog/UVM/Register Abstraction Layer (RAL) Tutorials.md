
## RAL Model

Многие цифровые контроллеры и блоки имеют регистры, которые можно установить программно (прошить). Использование этих регистров может управлять поведением устройства, конфигурировать его. Регистры часто упоминаются как **SFR (<u><b>Spectial Function Registers</b></u>)**.

Верификационный инженер также ответственен за проверку доступа к регистрам, их функциональности.

Программное обеспечение должно оперировать транзакциями, основанными на поддерживаемом протоколе для записи и чтения регистров, для этого нужны driver,  sequencer, sequence_item. RAL модель предоставляет коллекцию методов и правил, чтобы сделать работу инженера верификации легче.

Абстрактный уровень регистров предоставляет библиотеки, используемые чтобы создать memory-mapped модель для регистров тестируемого устройства.

UVM RAL предоставляет классы и методы для моделирования регистров и памяти тестового устройства.

*"RAL модель - абстрактная модель для регистров и памяти DUT"*

![[Pasted image 20241120133527.png]]

**Front door access** - доступ через интерфейс шины, требует времени для доступа к регистрам
**Back door access** - используется база данных симулятора для прямого доступа к регистрам *тестируемого устройства*

#### Преимущества UVM RAL модели

1. Переиспользование
2. Когда RAL модель интегрирована в верификационное окружение, инженер имеет доступ к регистрам через методы чтения и записи, внутри фактические транзакции будут осуществляться с помощью модели.
3. Определяет свод правил, которому удобно соответствовать
4. Значение сброса может быть установлено
5. Поскольку используется ООП, до регистра можно достучаться через имя, а методы могут быть вызваны напрямую
6. Предоставляется соединение с собирающими покрытие "контейнерами"
7. Есть встроенный метод, сравнивающий предсказание со значением
8. Внутренний sequencer управляет подачей сигналов 


## RAL Model Structre

Регистровая модель или RAL блоки включают в себя регистровые файлы, регистры, память, отображения и другие блоки

библиотека UVM содержит базовые классы для всего необходимого

| RAL blocks      | Corresponding RAL base classes |
| --------------- | ------------------------------ |
| memory          | `uvm_mem`                      |
| address map     | `uvm_reg_map`                  |
| register blocks | `uvm_reg_block`                |
| register files  | `uvm_reg_file`                 |
| registers       | `uvm_reg`                      |
| register fields | `uvm_reg_field`                |
![[Pasted image 20241120135229.png]]

Все классы унаследованы от `uvm_object` -> `uvm_void`


## RAL Classes

#### `uvm_reg_block`

Базовый для определенных пользователем регистровых блоков, содержит все регистры, отображения, регистровые файлы и другие регистровые блоки если необходимо. Блок может содержать несколько адресных карт, каждая соответствует своему интерфейсу. Регистровый блок находится на высшем уровне иерархии.

| Методы            | Описание                                                        |
| ----------------- | --------------------------------------------------------------- |
| `new`             | Создает новый экземпляр                                         |
| `configure`       | Конфигурирует определенный экземпляр                            |
| `create_map`      | Создает адресную карту в блоке                                  |
| `set_default_map` | Определяет стандартную адресную карту                           |
| `lock_model`      | Блокирует модель и строит адресную карту                        |
| `get_block`       | Получить подблоки                                               |
| `get_map`         | Получить адресную карту                                         |
| `get_registers`   | Получить регистры                                               |
| `get_fields`      | Получить поля                                                   |
| `get_memories`    | Получить память                                                 |
| `reset`           | Сбросить отображение для блока                                  |
| `update`          | Обновить отображаемые значения                                  |
| `set_backdoor`    | Установить пользовательский backdoor для всех регистров в блоке |
| `get_backdoor`    | Получить пользовательский backdoor для всех регистров в блоке   |
| `add_hdl_path`    | Добавить путь к HDL                                             |

```systemverilog
// Пример
class RegModel_SFR extends uvm_reg_block;
	// declaration for regs, blocks, reg files, mem, address map
	`uvm_object_utils(RegModel_SFR)
	function new(string name = "RegModel_SFR");
		super.new(name, ./*other arguments*/);
	endfunction

	virtual function void build();
		// create regs, blocks, reg files, mem, address map
	endfunction
endclass
```

#### `uvm_reg`

Базовый для пользовательского класса регистра. Подражает регистру устройства. Поля регистра представляются классом `uvm_reg_fields`.

| Методы         | Описание                                                        |
| -------------- | --------------------------------------------------------------- |
| `new`          | Создает новый экземпляр                                         |
| `configure`    | Конфигурирует конкретный экземпляр                              |
| `set_offset`   | Модифицирует сдвиг? регистра                                    |
| `get_offset`   | Возвращает сдвиг регистра                                       |
| `get_address`  | Возвращает базовый внешний физический адрес регистра            |
| `get_fields`   | Возвращает поля регистров                                       |
| `set`          | Устанавливает желаемое значение регистра                        |
| `get`          | Возвращает желаемое значение                                    |
| `reset`        | Сбрасывает желаемое/отражаемое значение регистра                |
| `update`       | Обновляет желаемое значение регистра                            |
| `read`         | Считать текущеще значение из регистра                           |
| `write`        | Записать определенное значение в регистр                        |
| `mirror`       | Обновить отражаемое значение для регистра                       |
| `set_backdoor` | Установить пользовательский backdoor для всех регистров в блоке |
| `get_backdoor` | Получить пользовательский backdoor для всех регистров в блоке   |
| `add_hdl_path` | Добавить путь к HDL                                             |

```systemverilog
class my_reg extends uvm_reg;
	// declaration for register fields
	`uvm_object_utils(my_reg)
	function new(string name = "my_reg");
		super.new(name, ./*other args*/);
	endfunction

	virtual function void build();
		// create and configure register fields
	endfunction
endclass
```

#### `uvm_reg_field`

Представляет собой поля регистра, каждый регистр может иметь одно или несколько полей, которые в свою очередь могут иметь разные политики доступа

| Политика<br>доступа | Описание                                                                                          |
| ------------------- | ------------------------------------------------------------------------------------------------- |
| RO                  | Read Only. Можно прочесть, запись не возымеет эффекта                                             |
| RW                  | Read Write. Может быть модифицировано и считано                                                   |
| W1C                 | Write 1 to Clear:<br>1. Запись 1 очищает бит<br>2. Запись 0 нет эффекта<br>3. Чтение значения     |
| W1S                 | Write 1 to Set:<br>1. Запись 1 устанавливает<br>2. Запись 0 нет эффекта<br>3. Считывание значения |

| Методы       | Описание                              |
| ------------ | ------------------------------------- |
| `new`        | Создание экземпляра                   |
| `configure`  | Конфигурирование экземпляра           |
| `set_access` | Установка политики доступа поля       |
| `get_access` | Получение политики доступа            |
| `set`        | Установка желанного значения          |
| `get`        | Получение необходимого значения       |
| `reset`      | Сбросить необходимое значение поля    |
| `read`       | Считать текущее значение из поля      |
| `write`      | Записать определенное значение в поле |
| `mirror`     | Обновить *mirrored* значение для поля |

#### `uvm_reg_file`

Регистровый файл - ничто иное как множество регистров или регистровых файлов.

| Методы         | Описание                           |
| -------------- | ---------------------------------- |
| `new`          | Создание экземпляра                |
| `configure`    | Конфигурирование экземпляра        |
| `get_regfile`  | Получить регистровый файл родителя |
| `add_hdl_path` | Добавляет путь к HDL               |

#### `uvm_mem`

UVM RAL также поддерживает реализацию памяти. Подобно тому, как `uvm_reg` "подражает" регистрам устройства, `uvm_mem` подражает памяти *DUT*.

| Методы         | Описание                                                            |
| -------------- | ------------------------------------------------------------------- |
| `new`          | Создание экземпляра                                                 |
| `configure`    | Конфигурация экземпляра                                             |
| `set_offset`   | Изменяет смещение в памяти                                          |
| `get_offset`   | Возвращает смещение в памяти                                        |
| `get_address`  | Возвращает базовый адрес ячейки памяти                              |
| `read`         | Читает текущее значение ячейки памяти                               |
| `write`        | Записывает определенное значение в ячейку памяти                    |
| `set_backdoor` | Устанавливает определенный пользователем <br>черный вход для памяти |
| `get_backdoor` | Получить user-defined backdoor для памяти                           |
| `add_hdl_path` | Добавляет путь к HDL                                                |

#### `uvm_reg_map`

Используется для доступа к модели через определенный интерфейс/шину.

| Методы              | Описание                                                   |
| ------------------- | ---------------------------------------------------------- |
| `new`               | Создание экземпляра                                        |
| `configure`         | Конфигурация экземпляра                                    |
| `add_reg`           | Добавить регистр                                           |
| `add_mem`           | Добавить память                                            |
| `add_submap`        | Добавить адрес отображения                                 |
| `reset`             | Сброс зеркал для всех регистров в этой адресной карте      |
| `set_sequencer`     | Установка секвенсора и связанного с ним адаптера для карты |
| `get_sequencer`     | Получение секвенсора и адаптера для карты                  |
| `get_registers`     | Получить регистры                                          |
| `get_reg_by_offset` | Получить регистр, отображенный по смещению                 |
| `get_mem_by_offset` | Получить памяти, отображенной по смещению                  |

```systemverilog
// Set the sequencer and associated adapter for a single map
reg_model.axi_map.set_sequencer(
	.sequencer(agt.sequencer),
	.adapter(adapter)
);

// Where,
// - reg_model - register block
// - axi_map - address map
// - adapter - register adapter for converting register 
//             transactions to bus transaction
```

## RAL methods in model

Важно понимать как методы `read`, `write`, `set`, `get`, `update`, `mirror`, и т.д. реагируют для mirrored/desired значения.

**Desired value** - значение которое должно быть запрограммированно в DUT с помощью тестового окружения.

**Mirrored value** - значение, хватающее значение регистра DUT, т.е. текущее состояние регистра DUT. Может быть изменено во время выполнения транзакций на шине.

#### RAL Methods

![[Pasted image 20241120182704.png]]

1. `write` метод записывает значение в регистр тестируемого устройства. Он также обновляет *желаемое* и *отраженное* значение.
2. `read` метод считывает значение регистра DUT и обновляет *желаемое* и *отраженное* значения.
3. `mirror` метод действует как метод `read`, в добавок он сравнивает значение с *отражаемым* значением если аргумент проверки = `UVM_CHECK`. Если *отражаемое* значение отличается от считанного с устройства, будет сообщение об ошибке. 
4. `get_mirrored_value` получает *отраженное* значение полей регистра, но не читает сам регистр в устройстве.
5. `set` метод обновляет *необходимое* значение и не записывает в регистр устройства, чтобы обновить *желаемое* значение в регистре устройства используется метод `update`.
6. `get` метод читает *желаемое* значение и не читает фактическое значение регистра устройства.
7. `update` метод обновляет *желаемое* значение в регистре устройства, если есть различия между *желаемым* и *отражаемым* значениями.
8. `predict` метод обновляет *отражаемое* и *желаемое* значения
9. `reset` метод сбрасывает *отражаемое* и *желаемое* значения


## RAL Adapter

RAL методы, типа `write` и `read` работают с транзакциями регистра (register sequence item), а DUT принимает или отправляет транзакции сигнального уровня (bus sequence item) от/к тестовому окружению через интерфейс. Следовательно, существует необходимость преобразования регистровых транзакций в транзакции на шине и наоборот. Этим занимается *Register Adapter*, пользовательский класс адаптер наследуется от базового класса `uvm_reg_adapter`.

![[Pasted image 20241121115910.png]]
Каждая операция front door доступа проходит через reg2bus и bus2reg API. Пользователь должен определить эти два метода:
```systemverilog
// reg2bus: конвертирует регистровую транзакцию в сигнальную
pure virtual function uvm_sequence_item reg2bus(
	const ref uvm_reg_bus_op rw
)

// bus2reg: конвертирует сигнальную транзакцию в регистровую
pure virtual function void bus2reg(uvm_sequence_item bus_item, 
								  ref uvm_reg_bus_op rw)
```

```systemverilog
// user-defined `reg_axi_adapter` implementation
class reg_axi_adapter extends uvm_reg_adapter;
	`uvm_object_utils(reg_axi_adapter)
	
	function new(string name = "reg_axi_adapter");
		super.new(name);
	endfunction

	virtual function uvm_sequence_item reg2bus(
		const ref uvm_reg_bus_op rw)
	);
		// ...
	endfunction

	virtual function void bus2reg(
		uvm_sequence_item bus_item,
		ref uvm_reg_bus_op rw
	);
		// ...
	endfunction
endclass
```

```systemverilog
// plug in register adapter in testbench environment
class env extends uvm_env;
	`uvm_component_utils(env)
	agent agt;
	reg_axi_adapter adapter;
	RegModel_SFR reg_model;  // top level register block

	function new(string name = "env", uvm_component parent = null);
		super.new(name, parent);
	endfunction

	function void build_phase(uvm_phase phase);
		super.build_phase(phase);
		agt = agent::type_id::create("agt", this);
		adapter = reg_axi_adapter::type_id::create("adapter");
		reg_model = RegModel_SFR::type_id::create("reg_model");
		reg_model.build();
		reg_model.reset();
		reg_model.lock_model();
		reg_model.print();

		uvm_config_db#(RegModel_SFR)::set(uvm_root::get(), "*",
										 "reg_model", reg_model);
	 endfunction

	function void connect_phase(uvm_phase phase);
		super.connect_phase(phase);
		reg_model.default_map.set_sequencer(
			.sequencer(agt.seqr),
			.adapter(adapter)
		);
		reg_model.default_map.set_base_addr('h0);
	endfunction
endclass
```


## RAL Predictor

UVM RAL Predictor - компонента, обновляющая отражаемое значение, основываясь на транзакциях в физическом интерфейсе, для которой UVM предоставляет базовый класс  `uvm_reg_predictor`. Регистры тестового устройства могут быть обновлены либо RAL методами (такими как `read` и `write`), либо запуском отдельных последовательностей с валидными адресами и данными для целевого агента, так что *driver* взаимодействует с *DUT* напрямую.

При front door доступе, UVM RAL предоставляет три модели для предиктора (предсказателя):
1. Implicit (Auto) Prefiction
2. Explicit Prefiction
3. Passive Prefiction

#### Auto Prediction

При автоматическом предсказании, методы front door доступа автоматически вызывают метод `predict` каждой транзакции, происходящей через шину, например чтение данных из регистра или записи записи в регистр в конце цикла синхронизации.

![[Pasted image 20241121123354.png]]
**Чтобы активировать автоматическое предсказание**: вызвать `set_auto_predict` метод
**Преимущество** - легкая реализация
**Недостаток** - не может обновить регистр модели, если регистровые *сиквенсы* написаны для доступа к регистрам *DUT*.

#### Explicit Prediction

Режим по умолчанию, который включает в себя явный внешний компонент predictior, который шпионит за транзакциями на шине и вызывает метод `predict`, чтобы обновить отражаемое регистром значение. Поскольку он напрямую взаимодействует с сигнальными транзакциями, ему необходим регистровый адаптер, чтобы конвертировать транзакции на шине в транзакции регистра.

![[Pasted image 20241121131630.png]]

```systemverilog
// create a predictor
uvm_reg_predictor #(axi_seq_item) axi_predictor;
axi_predictor = uvm_reg_predictor #(axi_seq_item)::type_id::create(
	"axi_predictor", this
);

// configure
axi_predictor.map = reg_model.default_map;  // assigning map handle
axi_predictor.adapter = axi_reg_adapter;  // assigning adapter handle

// connect
axi_mon.ap_mon.connect(axi_predictor.bus_imp);  // monitor analysis 
											   // port and predictor 
											   // analysis import 
											   // connection
```

**Преимущества:** в этом режиме модель регистра обновляется для выполнения транзакций по интерфейсу целевой шины
**Недостатки:** необходимо создавать, конфигурировать и соединять дополнительный компонент предиктора.

```systemverilog
// plugin reg_predictor in testbanch environment
class env extends uvm_env;
  `uvm_component_utils(env)
  agent agt;
  reg_axi_adapter adapter;
  uvm_reg_predictor #(axi_seq_item) axi_predictor;
  RegModel_SFR reg_model;  // top level register block

  function new(string name = "env", uvm_component parent = null);
    super.new(name, parent);
  endfunction

  function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    agt = agent::type_id::create("agt", this);
    adapter = reg_axi_adapter::type_id::create("adapter");
    axi_predictor = uvm_reg_predictor #(axi_seq_item)::type_id::create(
	  "axi_predictor", this
	);
	reg_model = RegModel_SFR::type_id::create("reg_model");
	//..
  endfunction

  function void connect_phase(uvm_phase phase);
    super.connect_phase(phase);
    reg_model.default_map.set_sequencer(
	  .sequencer(agt.seqr),
	  .adapter(adapter)
	);
	reg_model.default_map.set_base_addr('h0);
	axi_predictor.map = reg_model.default_map;  // assigning map handle
	axi_predictor.adapter = axi_reg_adapter;  // assign adapter handle
	axi_mon.ap_mon.connect(axi_predictor.bus_imp);  // monitor analysis
												   // port and predict
												   // analysis import
												   // connection
   endfunction
```

#### Passive Prediction

Пассивное предсказание - подобно явному, которое использует компонент предсказателя, за исключением того, что методы front door доступа к регистрам не могут быть использованы. 

![[Pasted image 20241121133303.png]]
**Преимущества:** этот режим держит регистровую модель обновленную для транзакций через целевой интерфейс шины
**Недостатки:** методы front door доступа к регистрам не могут быть использованы, только backdoor доступ возможен, поэтому, режим редко используется.


## UVM RAL model global declaration and built-in defines

UVM RAL модель устанавливает определенные типы, перечисления и дефайны глобально.

#### RAL Model Defines

| Defines                    | Описание                                                                                                        |
| -------------------------- | --------------------------------------------------------------------------------------------------------------- |
| ``UVM_REG_ADDR_WIDTH`      | Максимальная ширина адреса (в битах)<br>По умолчанию = 64<br>Используется чтобы определить тип `uvm_reg_addr_t` |
| ``UVM_REG_DATA_WIDTH`      | Максимальная ширина данных<br>По умолчанию - 64<br>Используется для определения `uvm_reg_data_t` типа           |
| ``UVM_REG_BYTENABLE_WIDTH` | -                                                                                                               |
| ``UVM_REG_CVR_WIDTH`       | Максимальное число бит в множестве моделей <br>покрытия `uvm_reg_cvr_t`<br>По умолчанию = 32                    |

#### RAL Model Types

| Типы                   | Описание                                                              |
| ---------------------- | --------------------------------------------------------------------- |
| `uvm_reg_addr_t`       | 2-state значение адреса с шириной ``UVM_REG_ADDR_WIDTH`               |
| `uvm_reg_addr_logic_t` | 4-state значение адреса с шириной ``UVM_REG_ADDR_WIDTH`               |
| `uvm_reg_data_t`       | 2-state значение данных шириной ``UVM_REG_DATA_WIDTH`                 |
| `uvm_reg_data_logic_t` | 4-state значение данных шириной ``UVM_REG_DATA_WIDTH`                 |
| `uvm_reg_byte_en_t`    | 2-state byte_enable значение с шириной<br>``UVM_REG_BYTENABLE_WIDTH1` |
| `uvm_reg_cvr_t`        | Значение модели покрытия шириной ``UVM_REG_CVR_WIDTH`                 |
| `uvm_hdl_path_slice`   | Фрагмент пути к HDL                                                   |

#### RAL Model Enumerations
| Перечисления | Описание                            |
| ------------ | ----------------------------------- |
| uvm_status_e | Возвращает статус регистра операций |


# Cookbook. Register Abstraction Layer

## The UVM Register Package

Модель регистра UVM предоставляет возможность отслеживать содержимое регистра Тестируемого Устройства (**ТУ**) и предоставляет удобный уровень доступа к регистру и ячейкам памяти внутри ТУ.

Абстракция модели регистра отражает структуру спецификации аппаратно-программного регистра.

#### Integrating A Register Model

При интеграции модели регистра в тестовое окружение, пре-реквизитами являются сама написанная модель регистра, а также класс адаптер для агента, который будет взаимодействовать с интерфейсной шиной ТУ.

В тестовом окружении конструируется объект модели регистра, а его дескриптор должен быть передан по окружению используя конфигурацию или механизм ресурсов (resource database).

Для того, чтобы подавать в агент из модели регистра, необходимо организовать связь между моделью и целевым `sequencer` так, чтобы при вызове `sequence` одного из методов `register model`, на необходимый `driver` отправлялся `sequence_item` сигнального уровня. `register model`, в свою очередь, постоянно обновляется в соответствии с состоянием аппаратного регистра с помощью `agent monitor` шины, а компонент `predictor` используется, чтобы преобразовывать сигнальную транзакцию агента в обновления в `register model`.  

![[Pasted image 20241122130508.png]]

#### Using A Register Model

После интеграции, модель регистра используется тестовым окружением, чтобы создавать стимулы используя `sequences` или с помощью компонентов анализа (`scoreboard`, мониторы функционального покрытия).

Модель регистра предназначена делать легче написание переиспользуемых последовательностей (`sequences`) доступа аппаратных регистров и областей памяти. Структура данных модели организована так, чтобы отражать иерархию ТУ, что дается легче написанием абстрактных и повторно используемых стимулов с точки зрения аппаратных блоков, памяти, регистров и полей, чем работу с низким уровнем абстракции. Модель содержит несколько методов доступа, которые используются последовательности для того, чтобы читать и записывать регистры. Эти методы приводят к преобразованию общих транзакций регистра в транзакции на целевой шине.
Пакет UVM содержит библиотеку встроенных тестовых последовательностей, которые могут быть использованы чтобы произвести большинство базовых тестов регистров и памяти, таких как проверка сброса значений регистра, проверка путей регистра и памяти. Эти тесты могут быть отключены для тех областей регистра или карты памяти, где они не имеют отношения (используются атрибуты регистра).

Частая форма стимулов - конфигурирование - когда устанавливаются регистры программируемого ТУ, чтобы включить определенный режим.
Регистровая модель поддерживает два способа доступа к регистрам ТУ: `front door` и `back door` доступ. `front door` доступ использует агент тестового окружения и доступ к регистрам осуществляется через протоколы передачи по шине. `back door` доступ использует доступ к базе данных симулятора, чтобы напрямую установить или пронаблюдать значение битов аппаратного регистра за нулевое время, пропуская интерфейсную логику.


## Register Model Overview

Регистровая модель реализуется при помощи пяти главных блоков - поле регистра - `register field`; память - `memory`;  регистровый блок - `register block`; карта регистра - `register map`. Поле регистра моделирует набор битов, соответствующих функции внутри регистра. Поле регистра будет иметь ширину и сдвиг позиции бита внутри регистра. Поле может иметь разные модификаторы доступа (чтение/запись, только чтение или только запись). Регистр содержит один или более полей. Блок регистра соответствует аппаратному блоку и содержит один или более регистров, а также одну или более регистровых карт.

Область памяти моделируется с помощью `uvm_mem`, который имеет диапазон, размер и содержится внутри регистрового блока, имеет свое смещение, определяемое регистровой картой.

Карта регистров определяет смещение адреса одного или более регистров или памяти в его родительском блоке с точки зрения конкретного интерфейса шины. Группа регистров может быть доступна из другого интерфейса под другим сдвигом адреса и это может быть смоделировано использованием другой адресной карты внутри родительского блока.

Регистровая модель UVM состоит из нескольких слоёв:
- `uvm_reg_field` - биты сгруппированные в соответствии с функцией внутри регистра
- `uvm_reg` - набор полей с различным битовым смещением
- `uvm_mem` - представляет собой блок памяти, который простирается в заданном диапазоне
- `uvm_block` - набор регистров (аппаратный блок) или подблоков с одной или более картами, может также содержать память.
- `uvm_map` - именованная адресная карта, со смещениями адресов регистров, памяти и подблоков. Также определяет целевой `sequencer` для доступа к регистрам из карты.

##### **Register Model Data Types**

Модель регистра использует несколько специальных типов данных, для того, чтобы стандартизировать ширину полей адреса и даных:

- `uvm_reg_data_t` - 64 бита, `'UVM_REG_DATA_WIDTH`, используется для полей данных регистра
- `uvm_reg_addr_t` - 64 бита, `'UVM_REG_ADDR_WIDTH` для переменных с адресами регистров

Оба типа основываются на типе `bit` в SystemVerilog, и, следовательно, имеют два состояния. По умолчанию они шириной в 64 бита, но ширину типа можно задать с помощью  переопределения `'define` в командной строке (не рекомендуется)

```bash
vlog +incdir+$(UVM_HOME)/src +define+UVM_REG_DATA_WIDTH=24
$(UVM_HOME)/src/uvm_pkg.sv
```

##### **Register Fields**

Слой низшего уровня - поле, соответствует одному или более битам внутри регистра. Каждое определение поле - создание экземпляра класса `uvm_reg_field`. Поля содержатся внутри класса `uvm_reg` и они конструируются и конфигурируются методом `configure()`:

```SystemVerilog
// uvm_field configure method prototype
fonction void configure(
	uvm_reg parent,  // the containing register
	int unsigned size,  // how many bits wide
	int unsigned lsb_pos,  // bit offset within the regisiter
	string access,  // RW, RO, WO
	bit volatile,  // Volatile if bit is updated by hardware
	uvm_reg_data_t reset,  // the reset value
	bit has_reset,  // wheter the bit is reset
	bit is_rand,  // whether the bit can be randomized
	bit individually_accessible  // i.e. totally containted within a byte lane
);
```

Метод конфигурирования используется как показано в примере регистра.
Когда поле создается, имя берется из переданной в метод `create` строки, которое по умолчанию такое же, как и дескриптор объекта.

##### **Registers**

Регистры моделируются расширением класса `uvm_reg` , с наполнением его объектами `field`. Общие характеристики регистра определяются его конструктором:

```SystemVerilog
// uvm_reg constructor prototype:
function new (
	string name="",  // register name
	int unsigned n_bits,  // register width in bits
	int has_coverage  // coverage model supported by the register
);
```

Класс `register` содержит `build` метод, используемый для создания и конфигурации полей. Следует отметить, что этот метод не вызывается в UVM build фазе, поскольку регистр - это `uvm_object`, а не компонента.

В следующем примере кода показано, как создается модель регистра CTRL в SPI master.

```systemverilog
class ctrl extends uvm_reg;
	`uvm_object_utils(ctrl);

	rand uvm_reg_field acs;
	rand uvm_reg_field ie;
	rand uvm_reg_field lsb;
	rand uvm_reg_field tx_neg;
	rand uvm_reg_field rx_neg;
	rand uvm_reg_field go_bsy;
	uvm_reg_field ;
	rand uvm_reg_field char_len;
	
	function new(string name = "ctrl");
		super.new(name, 14, UVM_NO_COVERAGE);
	endfunction

	virtual function void build();
		acs = uvm_reg_field::type_id::create("acs");
		ie = uvm_reg_field::type_id::create("ie");
		...
		reserved = uvm_reg_field::type_id::create("reserved");
		char_len = uvm_reg_field::type_id::create("char_len");

		acs.configure(this, 1, 13, "RW", 0, 1'b0, 1, 1, 0);
		ie.configure(this, 1, 11, "RW", 0, 1'b0, 1, 1, 0);
		...
		reserved.configure(this, 1, 7, "RO", 0, 1'b0, 1, 0, 0);
		char_len.configure(this, 7, 0, "RW", 0, 7'b0000000, 1, 1, 0);
		
	endfunction
endclass
```

Когда регистр добавляется в блок, он создается, вызывая создание и конфигурацию его полей, а затем он настраивается перед добавлением в регистровые карты, чтобы определить его сдвиг в памяти. Прототип метода конфигурации регистра `configure()`:

```systemverilog
function void configure(
	uvm_reg_block blk_parent,  // containing reg block
	uvm_reg_file regfile_parent = null,  // optional
	string hdl_path = ""  // used if hw reg can be specified in one path
);
```

##### **Memories**

Память моделируется наследованием от класса `uvm_mem`. Модель регистра относится к памяти как к областям или интервалам адресов, к которым можно обратиться. В отличие от регистров, значения памяти не хранятся из-за больших затрат.

Диапазон и тип доступа к памяти определяется конструктором:

```systemverilog
// uvm_mem constructor prototype:
function new(string name,  // name of the mem model
			 longint unsigned size,  // addr range
			 int unsigned n_bits,  // width of the mem
			 string access = "RW",  // access
			 int has_coverage = UVM_NO_COVERAGE);  // func coverage


// memory array 1 - size 32'h2000;
class mem_1_model extends uvm_mem;`
	`uvm_object_utils(mem_1_model);

	function new(string name = "mem_1_model");
		super.new(name, 32'h2000, 32, "RW", UVM_NO_COVERAGE);
	endfunction

endclass: mem_1_model
```

##### **Register Maps**

Регистровая карта предназначена для двух целей. Предоставляет информацию о смещениях регистров, памяти и блоков внутри них. Также карта используется, чтобы определить на каком агенте шины будут исполняться регистровые `sequences`, однако эта часть функциональности регистровых карт настраивается при интеграции регистровой модели в тестовое окружение UVM.

Для того, чтобы добавить регистр или память в карту, используются методы `add_reg()` или `add_mem()`. Их прототипы:

```systemverilog
// uvm_map add_reg method prototype:
function void add_reg(uvm_reg rg,  // reg object handle
					  uvm_reg_addr_t offset,  // addr offset
					  string rights = "RW",  // acces policy
					  bit unmapped = 0,  // if true, reg doesn't
									     // apear in the addr map
					  uvm_reg_frontdoor frontdoor = null);  // handle to reg
														   // fdoor access obj

// uvm_map add_mem method prototype:
function void add_mem(uvm_mem mem,  // mem object handle
					  uvm_reg_addr_t offset,  // mem addr offset
					  string rights = "RW",  // mem access policy
					  bit unmapped = 0,  // if true, mem isn't in addr map
								         // and a fdoor access needs tb definded
					  uvm_reg_frontdoor frontdoor = null);  // handle to mem
														   // fdoor access obj
```

Внутри блока может быть несколько регистровых карт, каждая из которых определяет разные адресные карты и разные интерфейсы шины.

##### **Register Blocks**

Этот класс может быть использован как контейнер для регистров и памяти, представляя регистры в уровне аппаратных функциональных блоков, или как контейнер множества блоков, представляющих регистры в аппаратной подсистеме или полностью организованной в блоках СнК. Для того чтобы определить смещения адресов регистров и памяти, блок содержит регистровую адресную карту. Регистровая карта должна быть создана внутри регистрового блока использованием метода `create_map()`:

```systemverilog
// prototype for the create_map method
function uvm_reg_map create_map(string name,  // name of the map handle
								uvm_reg_addr_t base_addr,  // the maps base addr
								int unsigned n_bytes,  // map access width in bytes
								uvm_endianness_e endian,  // the endianess of the map
								bit byte_addressing=1);  // wheter byte addressing en

// example:
AHB_map = create_map("AHB_map", 'h0, 4, UVM_LITTLE_ENDIATN);
```

Параметр *`n_bytes`* - размер слова (ширина шины) шины, которой карта соответствует. Если ширина регистра превышает ширину шины, более чем одно обращение к шине необходимо для чтения или записи регистра через такую шину. Аргумент *`byte_addressing`* влияет на то, как инкрементируется адрес при таком последовательном обращении. Например, если `n_bytes=4` и `byte_addressing=0`, то доступ к регистру шириной 64 бита и сдвигом 0 приведет к двум обращениям по шине с адресами 0 и 1. С `byte_addressing=1`, такой же доступ приведет к двум обращениям по адресам 0 и 4. 

Значение по умолчанию для `byte_addressing` - 1.

Первая карта, которая будет создана в регистровом блоке, присваивается элементу `default_map` регистрового блока.

Следующий пример кода показывает регистровый блок мастера SPI, он определяет дескриптор регистрового класса для каждого регистра внутри мастера, затем метод `build` создает и сконфигурирует каждый регистр, перед добавлением его в карту под соответствующим смещением:

```systemverilog
// spi_reg_block
class spi_reg_block extends uvm_reg_block;
	`uvm_object_utils(spi_reg_block)

	rand rxtx0 rxtx0_reg;
	rand rxtx1 rxtx1_reg;
	rand rxtx2 rxtx2_reg;
	rand rxtx3 rxtx3_reg;
	rand ctrl ctrl_reg;
	rand drivier driver_reg;
	rand ss ss_reg;

	uvm_reg_map APB_map;  // block map

	function new(string name = "spi_reg_block");
		super.new(name, UVM_NO_COVERAGE);

	endfunction

	virtual function void build();
		rxtx0_reg = rxtx0::type_id::create("rxtx0_reg");
		rxtx0_reg.configure(this, null, "");
		rxtx0_reg.build();

		...

		ctrl_reg = ctrl::type_id::create("ctrl_reg");
		ctrl_reg.configure(this, null, "");
		ctrl_reg.build();

		...

		ss_reg = ss:type_id::create("ss");
		ss_reg.configure(this, null, "");
		ss_reg.build();

		// Map name, Offset, Number of bytes, Endianess
		APB_map = create_map("APB_map", 'h0, 4, UVM_LITTLE_ENDIAN);

		APB_map.add_reg(rxtx0_reg, 32'h00000000, "RW");
		APB_map.add_reg(rxtx1_reg, 32'h00000004, "RW");
		...
		APB_map.add_reg(ss_reg, 32'h00000018, "RW");

		lock_model();
		
	endfunction
	
endclass
```

Финальное выражение в `build` методе - метод `lock_model()`, используется чтобы финализировать карту адресов и гарантировать то, что модель не сможет быть изменена другим пользователем.

**Hierarchical Register Block**

Регистровый блок из предыдущего примера может быть использован блочной верификации, но если SPI интегрирован в больший дизайн, регистровый блок SPI может быть комбинирован с другими регистровыми блоками в бОльшем блоке, чтобы создать регистровую модель. Группа блоков включает каждый подблок и добавляет их в новую адресную карту кластерного уровня. Этот процесс может быть замещен полной регистровой картой СнК, содержащей несколько вложенных уровней регистровых блоков. Следующий пример кода показывает как это должно быть сделано для подсистемы с SPI мастером и несколькими другими периферийными блоками.

```systemverilog
package pss_reg_pkg;
	import uvm_pkg::*;
	`include "uvm_macros.svh"

	import spi_reg_pkg::*;
	import gpio_reg_pkg::*;

	class pss_reg_block extends uvm_reg_block;
		`uvm_object_utils(pss_reg_block)

		function new(string name = "pss_reg_block");
			super.new(name);
		endfunction

		rand spi_reg_block spi;
		rand gpio_reg_block gpio;

		function void build();
			AHB_map = create_map("AHB_map", 0, 4, UVM_LITTLE_ENDIAN);

			spi = spi_reg_block::type_id::create("spi");
			spi.configure(this);
			spi.build();
			AHB_map.add_submap(this.spi.default_map, 0);

			gpio = gpio_reg_block::type_id::create("gpio");
			gpio.configure(this);
			gpio.build();
			AHB_map.add_submap(this.gpio.default_map, 32'h100);

			lock_model();
			
		endfunction: build
	endclass: pss_reg_block
endpackage: pss_reg_pkg
```


## Register Model Structure

Регистровая модель использует пять главных блоков - поле регистра, регистр, память, регистровый блок, регистровая карта. Регистровое поле моделирует коллекцию битов, соответствующих функции внутри регистра. У поля есть ширина и позиция битового сдвига внутри регистра. Поле может меть различные модификаторы доступа. Регистры содержат одно или больше полей. Регистровый блок соответствует аппаратному блоку и содержит один или более регистров. Регистровый блок также содержит одну или больше регистровых карт.

Область памяти в устройстве моделируется классом `uvm_mem`, у которого есть диапазон или размер. Он также содержится внутри регистрового блока и имеет сдвиг, определяемый регистровой картой. Область памяти может быть смоделирована только для чтения, только для записи, для записи и чтения, при этом все обращения выполняются по всей ширине поля данных. `uvm_memory` не содержит полей.

Регистровая карта определяет смещение адресного пространства одного и более регистров или памяти в его родительском блоке с точки зрения конкретного интерфейса шины. Группа регистров может быть доступна разным интерфейсам с разным набором адресных сдвигов, что может быть смоделировано использованием другой адресной карты внутри родительского блока. Адресная карта также используется, чтобы определить, какой агент шины использовать при доступе к регистру и какой адаптер использовать, чтобы конвертировать основные регистровые транзакции и транзакции целевого интерфейса (`sequence_items`).

![[Pasted image 20241125114636.png]]

##### Register Model Data Types

Модели регистров используют несколько специальных типов данных, чтобы стандартизировать ширину адреса и поля данных:
![[Pasted image 20241125115252.png]]

Оба типа основаны на типе SystemVerilog `bit`, следовательно, имеют два состояния. По умолчанию их ширина - 64 бита, но ширина каждого типа может быть определена `'define`, который может быть перегружен из командной строки.

```bash
vlog +incdir+$(UVM_HOME)/src +define+UVM_REG_DATA_WIDTH=24
$(UVM_HOME)/src/uvm_pkg.sv
```

Поскольку это переменная компиляции, необходимо перекомпилировать пакет, что также возымеет глобальный эффект на все модели регистров, компоненты и последовательности, использующие данный тип данных. Поэтому не стоит использовать эту возможность.

##### Register Fields

Поля - нижний уровень. Соответствуют одному или более битам внутри регистра. Происходят от класса `uvm_reg_field`, содержатся внутри `uvm_reg` и конструируются и конфигурируются методом `configure()`:

```systemverilog
// uvm_field configure method prototype
function void configure(
	uvm_reg parent,  // containing reg
	int unsigned size,  // how many bits wide
	int unsigned lsb_pos,  // bit offset within reg
	string access,  // RW, RO, WO
	bit volatile,  // volatile if bit is updated by hw
	uvm_reg_data_t reset,  // reset value
	bit has_reset,  // wheter the bit is reset
	bit is_rand,  // wheter the bi can be randomized
	bit individually_accessible  // i.e. contained within a byte 
);
```

##### Registers

Моделируются путем расширения `uvm_reg` класса, который является контейнером для объектов-полей. Общие характеристики регистра определяются его конструктором:

```systemverilog
// uvm_reg constructor prototype:
function new(
	string name="",       // register name
	int unsigned n_bits,  // register width in bits
	int has_coverage      // coverage model supproted by the reg
);
```

Класс регистра содержит метод `build`, используемый для создания и конфигурации полей (не вызывается в UVM build фазе, поскольку регистр это объект а не компонента)

... то же самое, что и в предыдущем разделе


## Complex Address Maps

В СнК отображение адресов регистров и памяти обычно сложнее, чем одна карта. Когда  в системе есть несколько `master`, тогда она зачастую наблюдаются различными регистровыми картами, т.е. один и тот же регистр будет отображаться различными адресами, в зависимости от мастера шины, обращающегося к нему. Другой распространенный случай - динамическая карта адресов.

#### Multiple Address Mapping

Если к пространству аппаратных регистров можно получить доступ с помощью более чем одного интерфейса шины, блок может содержать больше адресных карт, чтобы поддерживать альтернативы. В примере ниже созданы две адресные карты, в которые были добавлены память и регистры с разными смещениями:

```systemverilog
// memory sub-system (mem_ss) register & memory block
class mem_ss_reg_block extends uvm_reg_block;
	`uvm_object_utils(mem_ss_reg_block)

	function new(string name = "mem_ss_reg_block");
		super.new(name, build_coverage(UVM_CVR_ADDR_MAP));
	endfunction

	// mem array configuration registers
	rand mem_offset_reg mem_1_offset;
	rand mem_range_reg mem_1_range;
	rand mem_offset_reg mem_2_offset;
	rand mem_range_reg mem_2_range;
	rand mem_offset_reg mem_3_offset;
	rand mem_range_reg mem_3_range;
	rand mem_status_reg mem_status;

	// memories
	rand mem_1_model mem_1;
	rand mem_2_model mem_2;
	rand mem_3_model mem_3;

	// map
	uvm_reg_map AHB_map;
	uvm_reg_map AHB_2_map;

	function void build();
		mem_1_offset = mem_offset_reg::type_id::create("mem_1_offset");
		mem_1_offset.configure(this, null, "");
		mem_1_offset.build();
		...
		mem_status = mem_status_reg::type_id::create("mem_status");
		mem_status.configure(this, null, "");
		mem_status.build();

		mem_1 = mem_1_model::type_id::create("mem_1");
		mem_1.configure(this, "");
		...

		// create maps
		AHB_map = create_map("AHB_map", 'h0, 4, UVM_LITTLE_ENDIAN, 1);
		AHB_2_map = create_map("AHB_2_map", 
							   'h0,
							   4, 
							   UVM_LITTLE_ENDIAN, 
							   1);

		// add registers and memories to the AHB_map
		AHB_map.add_reg(mem_1_offset, 32'h0000_0000, "RW");
		AHB_map.add_reg(mem_1_range, 32'h0000_0004, "RW");
		...
		AHB_map.add_reg(mem_status, 32'h0000_0018, "RO");
		AHB_map.add_mem(mem_1, 32'hF000_0000, "RW");
		...
		
		// add registers and memories to the second map
		AHB_2_map.add_reg(mem_1_offset, 32'h0000_0000, "RW");
		AHB_2_map.add_reg(mem_1_range, 32'h0000_0004, "RW");
		...
		AHB_2_map.add_reg(mem_status, 32'h0000_0018, "RO");
		AHB_2_map.add_mem(mem_1, 32'hF000_0000, "RW");
		...

		lock_model();
		
	endfunction: build
endclass: mm_ss_reg_block
```

#### Dynamic Address Mapping

В СнК адресные карты могут быть изменены на ходу ради следующих целей:
- Безопасность - Регистры становятся недоступными в различных состояниях
- Переконфигурация - после обнаружения или повторного подключения аппаратного ресурса, адресная карта должна быть переконфигурирована
- Добавление нового хоста в систему, необходимость в новом представлении адреса в адресной карте
- Виртуализация - когда во время прерывания различные виртуальные машины должны захватить общие ресурсы

Регистровая карта UVM поддерживает динамическое обновление отображения позволяя пользователю разблокировать переконфигурируемый регистровый блок. После операции `unlock_model()`, функция `unregister()` может быть использована, чтобы удалить регистровую карту из блока, дополнительная регистровая карта может быть добавлена в блок, также содержимое карты регистра может быть разрегистрировано либо дополнено.

Поскольку модель регистра СнК имеет возможность поддерживать несколько разных конфигураций регистровых карт, которые могут меняться динамически в течение теста, регистровый блок СнК должен содержать функции `re-map`, чтобы сделать операцию легкой в использовании. 

#### Dynamically Removing/Re-Adding Registers and Memories

Предполагая, что мы используем определенный ранее `mem_ss_reg_block`, может существовать режим безопасности на уровне микросхемы, в котором определенные регистры или области памяти не будут доступны пользователям. Следующий пример показывает метод перехода в безопасный режим путем отключения отображения `mem3` и связанных с ним регистров, а затем другой способ выхода из из защищенного режима путем восстановления отображения `mem3` и его регистров.

```systemverilog
// memory sub-system (mem_ss) register & memory block
class mem_ss_reg_block extends uvm_reg_block;
	`uvm_object_utils(mem_ss_reg_block)

	function new(string name = "mem_ss_reg_block");
		super.new(name, build_coverage(UVM_CVR_ADDR_MAP));
	endfunction

	// removes mem3 and the registers associated with it
	// from the AHB_map
	function void security_demap();
		unlock_model();
		AHB_map.unregister(mem3_offset);
		AHB_map.unregister(mem3_range);
		AHB_map.unregister(mem3);
		lock_model();
		
	endfunction

	// re-instates mem3 and the registers associated
	// with it into the AHB_map
	function void security_remap();
		unlock_model();
		AHB_map.add_reg(mem_3_offset, 32'h0000_0010, "RW");
		AHB_map.add_reg(mem_3_range, 32'h0000_0014, "RW");
		AHB_map.add_mem(mem_3, 32'h0001_0000, "RW");
		lock_model();
		
	endfunction
endclass: mem_ss_reg_block
```

#### Dynamically Changing An Address Map

Следующий пример кода иллюстрирует метод изменения отображения адреса на новое, а также другой метод, позволяющий вернуть его к исходной конфигурации. Он, опять же, основан на добавлении методов к `mem_ss_reg_block`.

```systemverilog
// memory sub-system (mem_ss) register & memory block
class mem_ss_reg_block extends uvm_reg_block;
	...

	// remaps the contents of the AHB_map for host_B
	function void remap_for_host_B();
		unlock_model();
		unregister(AHB_map);  // unregisters the addr map
		AHB_map = null;  // delete the orig map

		// create and configure the new version of map
		AHB_map = create_map("AHB_map", 'h0, 4, UVM_LITTLE_ENDIAN, 1);

		// add registers and memories to the map
		AHB_map.add_reg(mem_1_offset, 32'h2000_0000, "RW");
		AHB_map.add_reg(mem_1_range, 32'h2000_0004, "RW");
		...
		AHB_map.add_reg(mem_status, 32'h2000_0018, "RO");
		AHB_map.add_mem(mem_1, 32'hE000_0000, "RW");
		...

		lock_model();

	endfunction

	function void remap_to_default();
		unlock_model();
		unregister(AHB_map);
		AHB_map = null;
		AHB_map = create_map("AHB_map", 'h0, 4, UVM_LITTLE_ENDIAN, 1);

		// add registers and memories to the AHB_map
		AHB_map.add_reg(mem_1_offset, 32'h0000_0000, "RW");
		AHB_map.add_reg(mem_1_range, 32'h0000_0004, "RW");
		...
		AHB_map.add_reg(mem_status, 32'h0000_0018, "RO");
		AHB_map.add_mem(mem_1, 32'hF000_0000, "RW");
		...
		lock_model();

	endfunction
endclass: mem_ss_reg_block
```


## Specifying Registers

Аппаратные функциональные блоки, соединенные с центральными процессорами, управляются через отраженные в памяти регистры. Каждый бит адресной карты в программе соответствует триггеру. Для того, чтобы управлять и взаимодействовать с устройством, ПО должно читать и записывать регистры, и поэтому описание регистра организовано с использованием абстракции, именуемой аппаратно-программным интерфейсом или описанием регистра.

Этот интерфейс распределяет адреса карты памяти входов и выходов регистру, определяемому мнемоникой. Каждый регистр может быть разбит на поля или группы индивидуальных битов, которым снова присваивается мнемоническое значение. Поле может одним битом, или шириной во весь регистр. Поля могут иметь разные модификаторы доступа, например, read-only, write-only, read-write or clear on read. Регистр может иметь зарезервированные поля, иными словами, биты в регистре, которые не используются, но могут быть использованы в последующих версиях устройства. Пример описания регистра - управляющий регистр из SPI мастер ТУ. 

![[Pasted image 20241129162922.png]]

Для заданного функционального блока может существовать множество регистров, каждый из которых имеет адрес, являющийся смещением от базового адреса блока. Для доступа к регистру, процессор будет читать или записывать по этому адресу. ПО инженер использует регистровую карту, чтобы определить как запрограммировать аппаратное устройство, для этого можно воспользоваться набором определенных утверждений, которые отображают имена регистров и полей в численные значения в заголовочном файле, чтобы он мог работать на более абстрактном уровне детализации. К примеру, если он хочет включить прерывания в SPI мастер, он может прочитать из регистра `CTRL`, затем, логически сложить значение с `IE`, а после записать результирующее значение обратно в `CTRL`:

```systemverilog
spi_ctrl = reg_read(CTRL) | IE;
reg_write(CTRL, spi_ctrl);
```

Адресная карта регистров для SPI мастера в таблице ниже. Заметьте, что управляющий регистр находится по адресу смещения 0x10 от базового адреса SPI мастера.

![[Pasted image 20241129164745.png]]


## The Register Layer Adapter

Методы доступа регистровой модели генерируют циклы чтения и записи на шине, используя общие регистровые транзакции. Эти транзакции должны быть подогнаны к транзакции целевого интерфейса шины. `adapter` должен быть двунаправленным, для того, чтобы конвертировать регистровые транзакции запросов в элементы последовательности шины, а также иметь он должен иметь возможность преобразовывать ответные элементы последовательности (`sequence items`) шины обратно в регистровые `sequence items`. Адаптер наследуется от базового класса `uvm_reg_adapter`.

![[Pasted image 20241202112437.png]]

#### Adapter Is Part Of An Agent Package

Адаптер - часть пакета агента. Адаптер регистра должен быть предоставлен как часть пакета агента целевой шины. 

Если вы создаете свой собственный агент шины, вам следует включить в него адаптер, чтобы позволить пользователем вашего пакета агента использовать его с регистровой моделью.

#### Implementing An Adapter

Обобщенный `register item` реализуется как структура, для того, чтобы минимизировать количество используемых ресурсов памяти. Структура определяется как тип `uvm_reg_bus_op` и содержит 6 полей:

- `addr (uvm_reg_addr_t)` - Адресное поле, 64 бита по умолчанию
- `data (uvm_reg_data_t)` - Прочитанные или записанные данные (64)
- `kind (uvm_access_e)` - `UVM_READ` или `UVM_WRITE`
- `n_bits (unsigned `int) - Количество переданных бит
- `byte_en (uvm_reg_byte_en_t)` - byte enable
- `status (uvm_status_e)` - `UVM_IS_OK`, `UVM_IS_X`, `UVM_NOT_OK`

Эти поля должны быть отражены к элементам последовательностей целевой шины, что делается путем расширения `uvm_reg_adapter` класса, который содержит два метода: `reg2bus()` и `bus2reg()`, которые должны быть перегружены. Класс адаптер также содержит два бита характеристик - `supports_byte_enable` и `provides_responses`, они должны быть установлены в соответствии с функциональностью, поддерживаемой целевой шиной и агентом целевой шины.

- `supports_byte_enable` - устанавливается в 1, если целевая шина и ее агент поддерживают `byte enables`
- `provides_responses` - устанавливается в 1, если драйвер целевого агента отправляет отдельные элементы ответов sequence_items, требующих обработки

Рассмотрим в качестве примера шину APB, `sequence_item` шины apb содержит 3 поля (`addr`, `data`, `we`), которые соответствуют адресу, данным и направлению шины. При преобразовании ответа APB шины в регистровую транзакцию поле статуса будет установлено в `UVM_IS_OK`, поскольку APB агент не поддерживает `SLVERR` бит статуса используется APB. 

Т.к. шина APB не поддерживает byte enables, `supports_byte_enable` бит установлен ноль внутри конструктора адаптера APB. 

`provides_responses` бит д. б. установлен, если драйвер агента возвращает отдельную ответную транзакцию из его транзакции запроса. Этот бит используется многоуровневым кодом модели регистра для определения того, нужно ли ждать ответа или нет, если бит установлен, а драйвер не возвращает ответа, генерация стимулов заблокируется.

Поскольку используемый APB драйвер возвращает `item_done()`, следовательно он использует один элемент последовательности для запроса и ответа, то `provides_responses` бит также устанавливается в 0 конструктором.

Пример кода для APB адаптера регистра:

```systemverilog
class reg2apb_adapter extends uvm_reg_adapter;
	`uvm_object_utils(reg2apb_adapter)
	function new(string name = "reg2apb_adapter");
		super.new(name);
		supports_byte_enable = 0;
		provides_responses = 0;
	endfunction

	virtual function uvm_sequence_item reg2bus(const ref uvm_reg_bus_op rw);
		apb_seq_item apb = apb_seq_item::type_id::create("apb");
		apb.we = (rw.kind == UVM_READ) ? 0 : 1;
		apb.addr = rw.addr;
		apb.data = rw.data;
		return apb;
	endfunction: reg2bus

	virtual function void bus2reg(uvm_sequence_item bus_item,
								 ref uvm_reg_bus_op rw);
		 apb_seq_item apb;
		 if (!$cast(apb, bus_item)) begin
			 `uvm_fatal(...)
			 return
		 end
		 rw.kind = apb.we ? UVM_WRITE : UVM_READ;
		 rw.addr = apb.addr;
		 rw.data = apb.data;
		 rw.status = UVM_IS_OK;
	 endfunction: bus2reg
	 
endclass: reg2apb_adapter
```

Код адаптера может быть найден в файле reg2apb_adapter.svh к директории /agents/apb_agent/

#### Burst Support

Методы доступа регистровой модели поддерживают единый доступ к регистрам, который соответствует обычной модели доступа к регистрам - доступ запрашивается индивидуально, а не как часть пакета. Если у вас есть регистры, которые необходимо протестировать на режим пакетного доступа, тогда рекомендуемый подход заключается в инициировании пакетной передачи данных сиквенсом, запущенным напрямую на целевом агенте шины.

Если вы используете режим пакетного доступа к регистрам, реализация предиктора должна быть способна идентифицировать пакеты и преобразовывать каждый такт в вызов `predict()` для соответствующего регистра. 

#### Common Adapter Issues

Адаптер - важное звено в цепи последовательностной коммуникации между методами доступа регистровой модели и драйвером. Ожидается, что API драйвера-секвенсора выполнено чисто и следует одной из стандартных моделей на стороне драйвера. Бит `provides_responses` должен быть установлен в 1, если ваш драйвер использует `put(rsp)`, чтобы вернуть ответ, и 0, если нет.

Если настройка `provides_responses` будет некорректной, случится одно из двух - либо генерация стимулов заблокируется, либо будет получен мгновенный ответ от `frontdoor` методов доступа с неверными ответами, а активность на шине произойдет позже.

Блокировка происходит, если вы используете модель завершения `get_next_item()`/`item_done()` и бит `provides_responses` установлен в 1 - причина тому то, что адаптер ждет ответа, который никогда не придет.

Мгновенный возврат случается при использовании модель завершения `get()`/`put(rsp)` и бит `provides_responses` установлен в 0, а вызов `get()` в коде драйвера моментально разблокирует метод завершения доступа секвенсора. Драйвер в таком случае продолжает выполнять запрос по шине перед тем, как вернуть ответ через метод `put(rsp)`. Адаптер игнорирует ответ. 

Вы также должны убедиться, что оба обращения (записи и чтения) следуют одной и той же модели завершения. В частности, убедитесь, что вы возвращаете ответ из драйвера для запроса записи, когда используете `get()/put(rsp)` модель завершения.

#### Context for the Bus

Если протокол шины, на который направлен адаптер, нуждается в большем количестве информации для правильной работы, существует метод для предоставления контекстной информации адаптеру.


## Integrating a UVM Register Model in a TestBench - Overview

#### Register Model Testbench Integration - Testbench Architecture Overview

Внутри тестового окружения UVM модель регистра используется либо как средство отображения текущего состояния аппаратного тестируемого устройства, либо как средство доступа к устройству напрямую или через интерфейс, а также обновление регистров базы данных модели.

Для тех компонентов или последовательностей, которые используют регистровую модель, модель должна быть сконструирована, а ее дескриптор передан, используя объект конфигурации или `resource`. Компоненты и последовательности, затем, могут использовать дескриптор модели регистра, чтобы вызывать методы доступа к данным, хранящимся внутри ее, или получать доступ к тестируемому устройству. 

Получить доступ к тестируемому устройству напрямую, модели регистра позволяют hdl пути, которые используются программами доступа к базе данных во время симуляции, для поиска аппаратных сигналов, соответствующих регистру. Модель регистра обновляется автоматически в конце каждого цикла прямого доступа. Путь, которым происходит обновление - вызов метода `predict()`, который обновляет отраженные значения регистров, к которым идет обращение. Для того, чтобы прямой доступ работал, дальнейшая интеграция с остальной структурой тестового окружения не требуется.

Модель регистра поддерживает frontdoor доступ к тестируемому устройству при помощи генерации общих регистровых транзакций, которые преобразуются в специфичную транзакцию целевого агента шины перед тем, как быть отправленной в секвенсор, и конвертируя любую возвращенную ответную информацию назад из транзакции агента в регистровую транзакцию. Это двунаправленное преобразование происходит внутри класса адаптера, который определен для целевой шины агента. На самом деле, есть только один способ, в котором сигнальная сторона доступа взаимодействует с тестовым окружением, но обновление или предсказание содержимого регистровой модели в конце такого доступа может происходить используя одну из трех моделей:

- Auto Prediction (автоматическое)
- Explicit Prediction (явное)
- Passive Prediction (пассивное)

##### Auto Prediction

Автоматическое предсказание - самый простой режим предсказания для доступа к регистру. В этом режиме различные методы доступа, вызывающие front door доступ автоматически вызывают метод `predict()`, либо используя данные, записанные в регистр, либо данные, прочитанные из регистра в конце цикла шины.

![[Pasted image 20241203114448.png]]

Такой режим наиболее прост в реализации, но страдает от недостатка, заключающегося в том, что он может держать регистровую модель обновленной только с трансферами, инициируемыми ей самой. Если любая другая последовательность напрямую обратятся к целевому секвенсору, чтобы обновить содержимое регистра, или если будет осуществлён доступ к регистру из других интерфейсов тестируемого устройства, тогда модель регистра останется не обновленной.

По умолчанию режим предсказания - явный. Для того, чтобы активировать автоматическое предсказание, используется метод `set_auto_predict()`. Также стоить заметить, что при использовании автоматического предсказания, если статус возвращает `UVM_NOT_OK`, регистровая модель не будет обновлена.

##### Explicit Prediction (Recommended Approach)

В режиме работы с явным прогнозированием внешний компонент предсказатель используется для прослушивания транзакций анализа агента целевой шины, а затем вызывается метод `predict()` регистра, к которому осуществлялся доступ, чтобы обновить отражаемое им значение. Компонент предиктор использует адаптер, чтобы преобразовывать транзакции анализа шины в транзакции регистра, затем он использует адресное поля, чтобы найти целевой регистр перед вызовом метода `predict()` с полем данных. Явное предсказание - режим прогнозирования по умолчанию, чтобы отключить его, используйте метод `set_auto_predict()`.

![[Pasted image 20241203114508.png]]

Главное преимущество использования явного предсказателя - он держит модель регистра обновленной в соответствии со всеми запросами, происходящими по целевой шине интерфейса. Конфигурация также предоставляет больше возможностей для поддержки вертикального повторного использования, когда доступ к ТУ может осуществляться от других агентов через шинный мост или сети межсоединений.

При вертикальном переиспользовании, окружение, которое поддерживает явное предсказание также может поддерживать пассивное предсказание как результат пере-конфигурации.

Также отметим, что при явном прогнозировании, значение статуса, возвращаемое в предсказатель, игнорируется. Это означает, что если статус ошибки (`UVM_NOT_OK`) возвращается при обращении к регистру, запрос к регистру необходимо будет отфильтровать, перед тем как отправлять информацию предсказателю, если эта передача с ошибкой не предназначена для обновления отражаемого значения регистра. Это может быть сделано в мониторе или модифицированного компонента предиктора тестового окружения, расположенного между монитором и предсказателем. 

##### Passive Prediction

В пассивном прогнозировании регистровая модель не принимает активного участия в доступе к ТУ, но сохраняет актуальность предиктором, когда какой-либо frontdoor запрос к регистру имеет место.

![[Pasted image 20241203124026.png]]

##### Vertically Integrated Environment Example

В интегрируемом примере окружения ниже, периферийный APB соединяется в AXI шиной с межсоединением через шинный мост. К устройству могут обратиться одно из двух ведущих устройств шины, одно из которых запускает последовательности напрямую, а другое использует регистровую модель, чтобы сгенерировать обращение.

![[Pasted image 20241203124627.png]]

В таком сценарии есть несколько мест, куда можно поместить предсказатель. Если он размещен в мастере агента, тогда обновление регистровой модели происходит только как результат доступа, который она сама генерирует. Если предиктор размещен в слейве агента, тогда предиктор сможет обновить регистровую модель при доступе от обоих ведущих устройств. Однако, если предиктор расположен на APB шине, он сможет подтверждать, что для периферийного устройства APB были использованы правильные отображения адресов и что сквозные передачи по шине выполняются корректно.

В этом примере также может выполняться до трех преобразований адресов и используемый предиктор должен использовать правильные регистровые модели карт, когда совершает вызов метода `predict()` целевого значения регистра.

##### Integrating a register model - Implementation Process

Рекомендуемый подход к реализации регистровой модели - использовать явное прогнозирование, поскольку у такого подхода есть несколько преимуществ, не последним из которых является то, что это облегчает вертикальное повторное использование.

Основываясь на предположении, что у вас есть регистровая модель, вам необходимо следовать шагам ниже в описанном порядке, чтобы интегрировать модель регистра в ваше окружение:

1. Понимать какая карта регистровой модели соответствует какому целевому агенту шины
2. Проверить есть ли у агента шины класс адаптер регистровой модели UVM, иначе, необходимо реализовать одну
3. Объявить и собрать регистровую модель в тесте, передавая ее дескриптор вниз по иерархии тестового окружения через конфигурацию (или ресурсы).
4. В каждом окружении, содержащем активный интерфейс шины, агент настраивает распределение шин по уровням.
5. В каждом окружении, содержащем агент шины, настроить предиктор

Регистровая модель будет содержать одну или более карт, определяющих отображения адресов регистров для конкретных интерфейсов шины. В большинстве случаев тестовые окружения блочного уровня нуждаются только в одной карте, но регистровые модели были разработаны для того, чтобы справиться с ситуацией, в которой ТУ имеет несколько интерфейсов шины, что может легко возникнуть в СнК. Каждая карта нуждается в слоях адаптора и предиктора.


## Integrating a UVM Register Model in a TestBench - Implementation

Процесс интеграции модели регистра включает в себя конструирование модели, размещение ее дескрипторов внутри конфигурационного объекта, а затем создание адаптационных слоёв.

#### Register Model Construction

Регистровая модель должна быть сконструирована в тесте, а ее дескриптор должен быть передан по остальной иерархии тестового окружения через объекты конфигурации. В случае окружения блочного уровня, передается дескриптор всей модели. Однако, в случае кластеров (под-систем) или окружения СнК, общая регистровая модель будет интеграцией регистровых блоков на подсистемы, а дескрипторы подсистем регистровых моделей будут разными для различных под-окружений.

Например, в случае блочного уровня тестового окружения для SPI, конфигурационный объект окружения будет содержать дескрипторы для регистровой модели SPI, которая будет создана и назначена в тесте. 

Следующий код взят из базового тестового класса для блочного тестового окружения SPI, заметьте, что регистровая модель создается с помощью фабрики, а затем вызывается метод `build()`.

> Следует помнить, что регистровая модель - не `uvm_component`, так что ее метод `build()` не будет вызван автоматически

```systemverilog
/*
 * From the SPI Test base class
 * 
 * Build the env, create the env configuration
 * including any sub configurations and assigning
 * virtual interface
 */
function void spi_test_base::build_phase(uvm_phase phase);
	// env configuration
	m_env_cfg = spi_env_config::type_id::create("m_env_cfg");
	// Register model
	// Enable all types of coverage available in the register model
	uvm_reg::include_coverage("*", UVM_CVR_ALL);
	// Create the register model:
	spi_rm = spi_reg_block::type_id::create("spi_rm");
	// Build and configure the register model
	spi_rm.build();
	// Assign a handle to the register model in the env config
	m_env_cfg.spi_rm = spi_rm;
	// etc.
endfunction: build_phase
```

В случае, где SPI - часть кластера, регистровая модель целого кластера (содержащая регистровую модель SPI как блок) будет создана в тесте, а затем объекту конфигурации окружения SPI будет передан дескриптор регистрового блока SPI как под-компонент общей регистровой модели:

```systemverilog
/*
 * From the build method of the PSS test base class:
 * 
 * PSS - Peripheral syb-system with a hierarchical 
 * register model with the handle pss_reg
 */
m_spi_env_cfg.spi_rm = pss_reg.spi;
```

#### Adaption Layer Implementation

Есть две части слоя адаптации регистра, первая часть реализует основанные на сиквенсах расслоение стимулов, а вторая часть реализует основанное на анализе обновление регистровой модели, при помощи компонента предиктора.

##### Register Sequence Adaption Layer

Слой адаптации регистровых последовательностей должен быть завершен в *connect* фазе UVM, когда известно, что регистровая модель и компоненты целевых агентов собраны.

Распределение регистров по уровням для каждого агента интерфейса целевой шины, поддерживаемого регистровой модель, должно выполняться единожды для каждой карты. В блочной среде это происходит в env, но окружение работает на более высоком уровне интеграции, это отображение следует совершить на верхнем уровне окружения. Для того, чтобы определить на верхнем ли уровне находится конкретный env, код должен проверять родителя данного регистрового блока - null или нет.

В следующем кода из метода `connect` компонента env блочного уровня SPI, APB_map из регистрового блока SPI накладывается на секвенсор агента APB, используя класс адаптер `reg2apb`  и метод `set_sequencer()` из объекта `APB_map`:

```systemverilog
/*
 * From the SPI env
 * 
 * Register layering adapter:
 */
reg2apb_adapter reg2apb;

function void spi_env::connect_phase(uvm_phase);
	if (m_cfg.m_apb.agent_cfg.active == UVM_ACTIVE) begin
		reg2apb = reg2apb_adapter::type_id::create("reg2apb");
		//
		// Register sequencer layering part:
		//
		// Only set up register sequencer layering if the top level env
		if (m_cfg.spi_rm.get_parent() == null) begin
			m_cfg.spi_rm.APB_map.set_sequencer(m_apb_agent.m_sequencer,
												reg2apb);
		end
	end
endfunction: connect_phase
```

##### Register Prediction

По умолчанию регистровая модель использует процесс под названием `explicit_prediction` для обновления данных регистра, основанный на завершении каждой транзакции чтения и записи, которая модель сгенерировала. Явное прогнозирование нуждается в использовании компоненты `uvm_reg_predictor`.

![[Pasted image 20241203164746.png]]

Компонент `uvm_reg_predictor` наследуется от `uvm_subscriber` и параметризуется типом транзакции анализа целевой шины. Он содержит дескрипторы адаптера шины целевого регистра и адресной карты регистровой модели, использованной  для взаимодействия с секвенсором агента шины. Он использует регистровый адаптер, чтобы преобразовывать транзакции анализа из монитора в регистровые транзакции, а затем ищет адрес регистра в соответствующей шине карте регистровой модели и меняет содержимое соответствующего регистра.

Компонент `uvm_reg_predictor` - часть библиотеки UVM и не нуждается в расширении. Однако, для его интегрирования, необходимо позаботиться о следующих вещах:

1. Объявить предиктор, используя sequence_item целевой шины как параметр класса.
2. Создать предиктор в методе `build()` окружения (env)
3. В методе `connect` - установить в карту предиктора карту регистра целевой регистровой модели
4. В методе `connect` - установить адаптер предиктора как класс адаптер целевого агента 
5. В методе `connect` - соединить экспорт анализ порт предиктора с портом анализа целевого агента

Предсказатель должен быть включен в каждое место, где есть монитор на целевой шине. Необходимый код показан ниже и во второй половине кода метода SPI connect.

```systemverilog
/*
 * From the SPI env
 * 
 * Register predictor:
 */
uvm_reg_predictor #(apb_seq_item) apb2reg_predictor;

function void spi_env::build_phase(uvm_phase phase);
	if (!uvm_config_db #(spi_env_config)::get(this, "", "spi_env_config", m_cfg)) begin
		`uvm_error("build_phase", "SPI env configuration object not found")
	end
	if (m_cfg.has_apb_agent) begin
		set_config_object("m_apb_agent*", "apb_agent_config",
							m_cfg.m_apb_agent_cfg, 0);
		m_apb_agent = apb_agent::type_id::create("m_apb_agent", this);
		// Build the regiser model predictor
		apb2reg_predictor = uvm_reg_predictor #(apb_seq_item)::type_id::create(
			"apb2reg_predictor", this
		);
	end
endfunction: build_phase

function void spi_env::connect_phase(uvm_phase phase);
	// Adapter Created in code shown above

	// Register prediction part:
	//
	// Replacing implicit registor model prediction with explicit one
	// based on APB bus activity observed by the APB agent monitor
	// Set the predictor map:
	apb2reg_predictor.map = m_cfg.spi_rm.APB_map;
	
	// Set the predictor adapter:
	apb2reg_predictor.adapter = reg2apb;
	
	// Connect the predictor to the bus agent monitor analysis port
	m_apb_agent.ap.connect(apb2reg_predictor.bus_in);
	
endfunction: connect_phase
```


## "Quirky" Registers

### Introduction

Quirky (необычные, причудливые) регистры - такие же как любые другие регистры, описанные с использованием базового класса регистра, за исключением одной вещи. У таких регистров есть специальное (quirky) поведение, которое либо не может быть описано, либо трудно описуемо, при использовании базового класса регистров. Базовый класс регистра может быть использован для описания поведения множества различных регистров - например, *clear-on-read* *(RC)*, *write-one-to-set* *(W1S)*, *write-zero-to-set (W0S)*. Эти встроенные модели поведения устанавливаются использованием атрибутов. Установка атрибутов вызывает встроенное поведение. Встроенные модели поведения могут быть использованы для большинства описаний регистров, но большинство верификационных окружений содержит небольшое число специальных регистров с моделью поведения, которое не может быть описано встроенными атрибутами. Такие регистры - quirky.

Примеры quirky регистров включают: *'clear on the third read'*, *'read a register that is actually a collection of bits from 2 other registers'*. Это регистры, у которых очень специфичное поведение. Необычные регистры находятся за пределами функциональности базового класса регистров и проще всего реализуются расширением функций базового класса, либо добавлением обратных вызовов.

Библиотека базовых классов регистров очень мощна и предлагает множество путей изменения модели поведения. Самые простые реализации расширения базового регистра или поля класса и переопределение ткущих виртуальных функций и тасков, таких как `set() get() read()` . В добавок, для замещения функций и задач могут быть добавлены обратные вызовы. Обратные вызовы вызываются в определенное время изнутри функций базового класса. Например, обратный вызов `post_predict()` вызывается из `uvm_reg_field::predict()`. Добавление этого обратного вызова позволяет значению поля изменяться (предсказываться). В IEEE 1800.2 LRM есть больше информации о перегрузке функций и тасков регистра, и об определении обратных вызовов.

### Quirky registers built-in to the library

Некоторые особенные модели поведения поставляются как часть библиотеки UVM - `uvm_reg_fifo.svh` и `uvm_reg_indirect.svh`. Эти классы реализуют поведение очереди и поведение косвенных регистров. Создание регистра fifo может быть совершено наследованием встроенного класса `uvm_reg_fifo`.

```systemverilog
class fifo_reg extends uvm_reg_fifo;
	function new(string name = "fifo_reg");
		super.new(name, 8, 32, UVM_NO_COVERAGE);
	endfunction
	
	`uvm_object_utils(fifo_reg)
endclass
```

### A Custom Quirky Register

Если поведение вашего quirky регистра не совпадает ни с какой из библиотечных, тогда вам нужно построить свой собственный. 

Пример ниже реализует регистр ID. Он возвращает элемент из списка при каждом последующем чтении. Каждое чтение возвращает следующий элемент списка. Когда достигается конец списка, возвращается первый элемент. Когда в регистр ID записываются данные, записанные данные заставляют указатель списка принимать значение, записанное в регистр. Например, запись 2 в регистр приведет к тому, что на следующем чтении будет возвращен третий элемент списка

##### **ID Register**

Вырезка некоторого кода, реализующего регистр ID приведена ниже.
```systemverilog
always @(posedge PCLK) begin
	if (PRESETn == 0) begin
		id_register_pointer <= 0;
		id_register_value <= '{'ha0, 'ha1, 'ha2, 'ha3, 'ha4,
								'ha5, 'ha6, 'ha7, 'ha8, 'ha9};
		current_value <= 32'ha0;
	end
	else begin
		if (PSEL & PENABLE) begin
			if (PWRITE) begin
				case (PADDR)
					// A write to the ID register overwrites the register pointer
					'ID: begin
						if (PWDATA < 10) begin
							id_register_pointer <= PWDATA[3:0];
						end
						else begin
							id_register_pointer < 0;
						end
					end
					'R_W: W_reg <= PWDATA;
				endcase
			end
			else begin
				// A read from the ID register advances the id_register_pointer
				if (PADDR == 'ID) begin 
					if (id_register_pointer == 9) begin
						id_register_pointer <= 0;
					end
					else begin
						id_register_pointer <= id_register_pointer + 1;
					end
				end
			end
		end
		if (~PENABLE) begin
			current_value <= id_register_value[id_register_pointer];
		end
	end
end

always @(*) begin
	if (PSEL) begin
		case (PADDR)
			// Return the current_value of the id register
			`ID: PRDATA = current_value;
			`R_W: PRDATA = R_reg;
			default: PRDATA = 0;
		endcase
	end
	else begin
		PRDATA = 0;
	end
end
```

##### **ID Register Model**

Реализация модели регистра ID приведена ниже. Регистр схож с обыкновенными регистрами, за исключением того, что он использует новый тип поля - `id_register_field`.

Поле `id_register_field` реализует особенную функциональность ID регистра в данном случае.
```systemverilog
// The ID Register.
// Just a register which has a special field - the
// ID Register field.
class id_register extends uvm_reg;
	id_register_dield F1;

	function new(string name="id_register");
		super.new(name, 8, UVM_NO_COVERAGE);
	endfunction

	virtual function void build();
		F1 = id_register_field::type_id::create("F1",, get_full_name());
		F1.configure(this, 8, 0, "RW", 0, 8'ha0, 1, 0, 1);
	endfunction

	`uvm_object_utils(id_register)

endclass
```
ID регистр конструирует поле ID внутри метода `build()`, как и другие регистры

##### ID Register Model Field

Поле регистра ID является простой реализацией требуемой функциональности. Методы обратного вызова `post_read()` и `post_write()` позволяют моделировать функциональность поля регистра.

Строго говоря, поле ID не моделирует поведение регистра ID, оно проверяет реализацию регистра в тестовом устройстве.

```systemverilog
// The ID Register (field).
// Each successive 'read' operation returns the next item
// from a list. When the end of the list is reached, it wraps
// around to the beginning.
// This list is a0, a1, ..., a9. (10 values)
class id_register_field extends uvm_reg_field;
	`uvm_object_utils(id_register_field)

	int id_register_pointer = 0;
	int id_register_pointer_max = 10;
	int id_register_value[] = 
		'{'ha0, 'ha1, 'ha2, 'ha3, 'ha4,
		  'ha5, 'ha6, 'ha7, 'ha8, 'ha9};

	int current_value;

	function new(string name = "id_register_field");
		super.new(name);
		current_value = id_register_value[0];
	endfunction: new

	task post_read(uvm_reg_item rw);
		if (value != current_value) begin
			`uvm_error(...)
		end
		id_register_pointer++;
		if (id_register_pointer >= id_register_pointer_max) begin
			id_register_pointer = 0;
		end
		current_value = id_register_value[id_register_pointer];
	endtask

	task post_write(uvm_reg_item rw);
		id_register_pointer = value;
		if (id_register_pointer >= id_register_pointer_max) begin
			id_register_pointer = 0;
		end
		current_value = id_register_value[id_register_pointer];
	endtask

endclass
```

Когда вызывается метод `post_read()`, он проверяет, что текущее значение поля `current_value` такое же, что и значение, прочитанное из ТУ. Затем он обновляет текущее значение поля в соответствии с инкрементированным значением указателя `id_register_pointer`, готовым к следующему чтению регистра.  

Метод `post_write()` записывает значение в указатель `id_register_pointer` и устанавливает `current_value` для следующего цикла чтения.

Поле ID регистра также содержит специальные значения, которые и будут прочитаны. Эти значения могут быть получены извне, или могут быть установлены из какого-то другого места.


## Register Model Coverage

### Controlling the build/inclusion of covergroups in the register model

Какие группы покрытия встраиваются внутрь блока регистров или объекта регистра определяется локальной переменной `m_has_cover`. Эта переменная имеет тип `uvm_coverage_model_e` и она должна быть инициализирована вызовом `build_coverage()` внутри конструктора объекта регистра, который присваивает значение ресурса `include_coverage` параметру `m_has_cover`. После того как эта переменная установлена, группы покрытия внутри объекта регистровой модели должны быть сконструированы в соответствии с категорией покрытия, к которой они относятся.

Конструирование различных групп покрытия основано на результате вызова `has_coverage()`.

По мере создания каждой категории *covergroup* переменная `m_cover_on` должна быть установлена на разрешение сборки покрытия для этого набора групп, это необходимо сделать вызовом метода `set_coverage()`.

### Controlling the sampling of covergroups in the register model

В зависимости от того, должна ли группа покрытия сэмплироваться автоматически во время доступа к регистру, либо как результат внешнего вызова `sample_values()`, два различных метода должны быть реализованы для каждого объекта. Метод `sample()` вызывается автоматически для каждого регистра и регистрового блока на любое обращение. Метод `sample_values()` предназначен для вызова из других мест тестового окружения. Оба этих метода проверяют битовое поле `m_cover_on`, используя метод `has_coverage()`, чтобы определить, следует или нет выполнять выборку набора групп покрытия .

### The register model coverage control methods

Здесь кратко описаны различные методы используемые для управления созданием групп покрытия и их влияние:

| Method                                            | Description                                                                                                                                                           |
| ------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Overall Control**                               |                                                                                                                                                                       |
| `uvm_reg::include_coverage(uvm_coverage_model_e)` | Статический метод, устанавливающий ресурс под ключом "include_coverage". Используется для управления собираемых регистровой моделью типов покрытия                    |
| **Build Control**                                 |                                                                                                                                                                       |
| `build_coverage(uvm_coverage_model_e)`            | Используется в конструкторе регистра для установки локальной переменной `m_has_cover` в значение, которое хранится в базе данных ресурсов по ключу "include_coverage" |
| `has_coverage(uvm_coverage_model_e)`              | Возвращает `true` если переданный тип покрытия включен в поле `m_has_cover`                                                                                           |
| `add_coverage(uvm_coverage_model_e)`              | Позволяет добавить переданный аргументом тип покрытия в поле `m_has_cover`                                                                                            |
| **Sample Control**                                |                                                                                                                                                                       |
| `set_coverage(uvm_coverage_model_e)`              | Включает сбор покрытия для переданного типа покрытия (по умолчанию не включен)                                                                                        |
| `get_coverage(uvm_coverage_model_e)`              | Возвращает `true` если сбор покрытия переданного типа включен                                                                                                         |
**Обратите внимание: невозможно установить включение покрытия поля до тех пор пока соответственно не будет установлено поле создания сбора покрытия (?)**

Значения перечисления `uvm_coverage_model_e`:

| uvm_coverage_model_e enum value | Description                                                     |
| ------------------------------- | --------------------------------------------------------------- |
| `UVM_NO_COVERAGE`               | Покрытие не включено                                            |
| `UVM_CVR_REG_BITS`              | Покрытие включено для битов регистра                            |
| `UVM_CVR_ADDR_MAP`              | Покрытие включено для индивидуального регистра и адресов памяти |
| `UVM_CVR_FIELD_VALS`            | Покрытие включено для полей регистра                            |
| `UVM_CVR_ALL`                   | Все типы покрытия регистра включены                             |
### An example

Следующий пример взят из реализации регистровой модели, включающей в себя функциональное покрытие.

В тесте должно быть включено покрытие модели регистра в целом для всего тестового окружения при помощи статического метода `uvm_reg::include_coverage()`:

```systemverilog
//
// Inside the test build method:
//
function void spi_test_base::build();
	uvm_reg::include_coverage(UVM_CVR_ALL);  // All register coverage types enabled
	//
	// ..
```

Первый отрывок кода - для групп покрытия, предназначенных для использования на блочном уровне для получения покрытия доступа чтения и записи блока регистра. Группа покрытия была обернута в класс, включенный в пакет регистровой модели, так с ней проще работать.

```systemverilog
//
// A covergroup (wrapped in a class) that is designed to get the
// register map read/write coverage at the block level
//
//
// This is a register access covergroup within a wrapper class
//
// This will need to be called by the block sample method
//
// One will be needed per map
//
class SPI_APB_reg_access_wrapper extends uvm_object;
	`uvm_object_utils(SPI_APB_reg_access_wrapper)

	covergroup ra_cov(string name) with function sample(uvm_reg_addr_t addr,
													   bit is_read);
		option.per_instance = 1;
		option.name = name;
		
		// To be generated:
		//
		// Generic form for bins is:
		//
		// bins reg_name = {reg_addr};
		ADDR: coverpoint addr {
			bins rxtx0 = {'h0};
			bins rxtx1 = {'h4};
			bins rxtx2 = {'h8};
			bins rxtx3 = {'hc};
			bins ctrl = {'h10};
			bins divider = {'h14};
			bins ss = {'h18};
		}

		// Standard code - always the same
		RW: coverpoint is_read {
			bins RD = {1};
			bins WR = {0};
		}

		ACCESS: cross ADDR, RW;
	
	endgroup: ra_cov
	
	function new(string name = "SPI_APB_reg_access_wrapper");
		ra_cov = new(name);
	endfunction: new
	
	function void sample(uvm_reg_addr_t offset, bit is_read);
		ra_cov.sample(offset, is_read);
	endfunction: sample
	
endclass: SPI_APB_reg_access_wrapper
```

Второй отрывок кода - для блока регистра, который включает группу покрытия. Код был упрощен, чтобы показать части, ответственные за управление моделью покрытия.

В конструкторе блока вызывается `build_coverage()`, это производит (конъюнкцию) аргумента перечисления покрытия, вместе с общей настройкой покрытия тестового окружения, которая была установлена методом `uvm_reg::include_coverage()` и поле `m_has_cover` настраивается в соответствии с результатом.

В методе `build()` блоков, метод `has_coverage()` используется для проверки включена ли модель покрытия, используемая в запрашиваемом блоке покрытия. Если это так, запрашиваемая группа покрытия собирается, а затем сбор покрытия для этой модели покрытия устанавливается, с помощью метода `set_coverage()`.

В методе `sample()` используется метод `get_coverage()` для проверки разрешен ли сбор покрытия данной модели, и только в том случае покрытие собирается для данной группы, передавая аргументы `address` и `is_read`.

```systemverilog
//
// The relevant parts of the spi_rm register block:
//

//--------------------------------------
// spi_reg_block
//--------------------------------------
class spi_reg_block extends uvm_reg_block;
	`uvm_object_utils(spi_reg_block)

	rand rxtx0 rxtx0_reg;
	rand rxtx1 rxtx1_reg;
	rand rxtx2 rxtx2_reg;
	rand rxtx3 rxtx3_reg;
	rand ctrl ctrl_reg;
	rand divider divider_reg;
	rand ss ss_reg;

	uvm_reg_map APB_map;  // block map

	// wrapped APB register access covergroup
	SPI_APB_reg_access_wrapper SPI_APB_access_cg;

	// new
	function new(string name = "spi_reg_block");
		// build_covaregs ANDs UVM_CVR_ADDR_MAP with the value set
		// by the include_coverage() call in the testbench
		// The result determines which coverage categories can be
		// built by this region of the register mdoel
		super.new(name, build_coverage(UVM_CVR_ADDR_MAP));
	endfunction: new

	// build
	virtual function void build();

		// Check that the address coverage is enabled
		if (has_coverage(UVM_CVR_ADDR_MAP)) begin
			SPI_APB_access_cg = SPI_APB_reg_acces_wrapper::type_id::create(
				"SPI_APB_access_cg"
			);
			// Enable sampling on address coverage
			set_coverage(UVM_CVR_ADDR_MAP);
		end

		// 
		// Create, build and configure the registers ...
		//
		
		APB_map = create_map("APB_map", 'h0, 4, UVM_LITTLE_ENDIAN);
		APB_map.add_reg(rxtx0_reg, 32'h00000000, "RW");
		APB_map.add_reg(rxtx1_reg, 32'h00000004, "RW");
		APB_map.add_reg(rxtx2_reg, 32'h00000008, "RW");
		APB_map.add_reg(rxtx3_reg, 32'h0000000c, "RW");
		APB_map.add_reg(ctrl_reg, 32'h00000010, "RW");
		APB_map.add_reg(divider_reg, 32'h00000014, "RW");
		APB_map.add_reg(ss_reg, 32'h00000018, "RW");
		
		add_hdl_path("DUT", "RTL");
		lock_model();
		
	endfunction: build

	// Automatically called when a block access occurs:
	function void sample(uvm_reg_addr_t offset, bit is_read, uvm_reg_map map);
		// Check wheterh coverage sampling is enabled for address accesses:
		if (get_coverage(UVM_CVR_ADDR_MAP)) begin
			// Sample the cg if access is for the APB_map
			if (map.get_name() == "APB_map") begin
				SPI_APB_access_cg.sample(offset, is_read);
			end
		end
	endfunction: sample

endclass: spi_reg_block
```


## Register-Level Stimulus

### Stimulus Abstraction

Стимулы, которые обращаются к регистрам, отображенным в памяти, должны быть как можно более абстрактными. Причиной тому то, что это:
- Упрощает разработку записи 
- Упрощает понимание для пользователей
- Предоставляет защиту против изменений карты регистров в течение цикла разработки
- Делает стимулы более легкими для переиспользования

Конечно, можно написать стимулы, которые совершают чтение и запись регистров напрямую через элемент последовательности агента с тяжело закодированными адресами и значениями (например, `read(32'h1000_f104); write(32'h1000_f108, 32'h05);`) - но эти стимулы придется переписывать, если базовый адрес тестируемого устройства изменился и должен быть декодирован с использованием спецификации поддержки кода регистра.

Регистровая модель содержит информацию, которая связывает на имена регистров с их адресами и поля регистра с битовыми позициями внутри регистра. Это означает, что регистровая модель облегчает запись стимулов, что уже является более высоким уровнем абстракции, например `read(SPI.ctrl);` или `write(SPI.rxtx0, 32'h0000_5467);`.

Модель регистра позволяет пользователям обращаться к полям регистра по имени. Например, если у вас есть модель регистра SPI с указателем `spi_rm` и вы хотите обратиться к управляющему регистру `ctrl`, то путь к нему - `spi_rm.ctrl`. Если вы хотите обратиться к полю `go_bsy` внутри управляющего регистра, то путь к нему - `spi_rm.ctrl.go_bsy`.

Регистровая модель интегрируется с агентом шины в тестовом окружении UVM. Для того, кто подает стимулы это означает, что он использует методы регистровой модели для инициации передач в/из регистра через интерфейс шины, вместо того, чтобы использовать последовательности, генерирующие `sequence_items` целевого интерфейса шины. Для создателя стимулов это уменьшает общее количество знаний, необходимых для того, чтобы начать работу.

### UVM Register Data Value Tracking

У регистровой модели есть собственная база данных, предназначенная для представления состояния аппаратных регистров. Для каждого регистра есть отражаемое значение (mirrored) и желаемое (desired) значение. Желаемое значение представляет состояние, которое регистровая модель в будущем использует для обновления устройства, но еще не использовала. Другими словами желаемое значение позволяет пользователю устанавливать индивидуальные поля регистров перед непосредственной записью. Отражаемое значение представляет текущее известное состояние регистра аппаратуры. Отражаемое значение обновляется в конце фронта циклов чтения и записи на шине, основываясь на значении данных регистровой модели (auto-prediction) либо на траффике, наблюдаемым монитором и отправленным предиктору для обновления содержимого регистровой модели (рекомендуемый подход интегрирования регистровой модели). Доступ backdoor обновляет модель автоматически. Отражаемое значение может устареть если какие-нибудь биты внутри изменчивы, то есть могут быть изменены аппаратными событиями, а не в результате программирования.

### UVM Register Access Methods

У регистровой модели есть число методов, которые могут быть использованы для чтения и записи регистров ТУ. Эти методы используют желаемое и отражаемое значения, чтобы идти в ногу с фактическим содержимым аппаратного регистра.

Регистровая модель может обращаться к аппаратным регистрам через переднюю либо заднюю дверь. Доступ через переднюю дверь (front door) происходит с использованием интерфейса шины и агента и симулирует транзакцию на шине в реальной жизни с соответствующими таймингами и событиями. Доступ через черный ход (back door) обращается к рутинам базы данных симулятора, чтобы получить текущее значение битов аппаратного регистра или назначить им определенное значение. Back door доступ происходит за 0 единиц времени симуляции

#### `read` and `write`

Метод `read()` возвращает значение аппаратного регистра. При использовании front door доступа, вызов `read()` приводит к передаче на шине, а желаемое и отражаемое значения регистровой модели обновляются предиктором шины по завершении цикла считывания.

```systemverilog
//
// read task prototype
//
task read(output uvm_status_e status,
		  output uvm_reg_data_t value,
		   input uvm_path_e path = UVM_DEFAULT_PATH,
		   input uvm_reg_map map = null,
		   input uvm_sequence_base parent = null,
		   input int prior = -1,
		   input uvm_object extension = null,
		   input string fname = "",
		   input int lineno = 0);

//
// Example = from within a sequence
// 
// Note use of positional and named arguments
// 
spi_rm.ctrl.read(status, read_data, .parent(this));
```

Метод `write()` записывает указанные значения в целевой аппаратный регистр. При front door доступе отражаемые и желаемые значения обновляются предиктором шины в конце цикла записи.
![[Pasted image 20250311121551.png]]
```systemverilog
// 
// write task prototype
//
task write(output uvm_status_e status,
		   input uvm_reg_data_t value,
		   input uvm_door_e path = UVM_DEFAULT_DOOR,
		   input uvm_reg_map map = null,
		   input uvm_sequence_base_parent = null,
		   input int prior = -1,
		   input uvm_object extension = null,
		   input string fname = "",
		   input int lineno = 0);

// 
// example - from within a sequence
//
// note use of positional and named arguments
//
spi_rm.ctrl.write(status, write_data, .parent(this));
```

Хотя методы чтения и записи могут использоваться как на регистровом, так и на уровне полей, их следует использовать только на уровне регистров, чтобы получать предсказуемые и переиспользуемые результаты. Чтение и запись уровня поля может работать только если поле занимает всю линию байт, когда целевая шина поддерживает доступ на уровне байтов. В то время как это может работать с регистровыми стимулами, написанными с мыслью об одном протоколе, если аппаратный блок интегрируется в подсистему, которая использует протокол шины, который не поддерживает byte_enable, тогда стимулы могут перестать работать.

Методы доступа `read()` и `write()` могут также использоваться для back door доступа, и тогда они завершаются и обновляют отображаемое значение мгновенно.

#### `set` and `get`

Методы `set()` и `get()` оперируют лишь с регистровой моделью и не обращаются к аппаратуре.

Вызов `set` назначит внутреннее "желаемое значение" регистру или полю регистровой модели. Это внутреннее значение будет функцией: значения аргумента, переданного `set`, текущего отражаемого значения, и модификатора доступа. Вызов `get` после `set` вернет вычисленное желаемое значение, которое может и не быть значением, переданным при вызове `set`.

Некоторые примеры:
- Для модификатора доступа RW вычисленное желаемое значение - всегда значение переданного в `set` аргумента, текущее отображаемое значение не имеет значения.
- Для W1C желаемое значение - побитовое И инвертированного значение аргумента и текущего отображаемого значения.
- Для RO желаемое значение - всегда текущее отображаемое значение.
- Для WC желаемое значение всегда 0Б независимо от значения аргумента и отображаемого значения.
Если `set` не приводит к тому, что желаемое значение отличается от текущего отображаемого, тогда полю нет необходимости обновляться. Если ни одно поле взятого регистра не должно обновиться, то и весь регистр не должен обновляться, следовательно, вызов `update` для этого регистра не приведет к активности на шине. 

Следует заметить, что множественные `set` в регистр или поле кумулятивны до момента вызова `update`. В норме регистр или полу устанавливается единожды перед вызовом `update`. Но в случаях, когда это не так, регистровая модель использует модификаторы доступа для обновления желаемого значения каждый раз. Когда вызывается метод `update`, финальное желаемое значение назначается на шину, если необходимо.

Например, для 16-битного W1T (write 1 to toggle) регистра:
```systemverilog
my_reg.set(16'hFFFF);  // desired is now ~mirrored
my_reg.set(16'hFFFF);  // desired is now that mirrored was originally
my_reg.update(...);
```

Это не приведет к какой-либо активности на шине, потому что вычисленное желаемое значение - отражаемое значение, переключенное дважды, т. е. совокупный результат такой же как и отражаемое значение, и следовательно, нет необходимости обновлять ТУ.

В следующей таблице приведена краткая информация о поведении операции `set`/`get` регистра в зависимости от режима доступа к каждому полю, устанавливаемого значения и текущего отображаемого значения.

- Для операций `set`/`update` над регистрами, таблица применяется к модификатору доступа каждого поля независимо.
- Управление шиной необходимо, если хотя бы одно из вычисленных желаемых значений полей отличается от их текущего отражаемого значения.
- Если происходит множественное присваивание перед обновлением, отображаемое значение представляет собой вычисленное желаемое значение

#### UVM Register `set`/`update` behavior vs access mode

| Access Mode      | Set<br>value<br>(A) | Mirror<br>value<br>(B) | Desired<br>value<br>(D) | Bus Access? | Value seen on<br>Bus during update() |
| ---------------- | ------------------- | ---------------------- | ----------------------- | ----------- | ------------------------------------ |
| RO, RC, RS       | x                   | x                      | no chg                  | N           | -                                    |
| RW, WRC, WRS, WO | A                   | B                      | A                       | if (B!=A)   | D                                    |
| WC, WCRS, WOC    | x                   | B                      | 0                       | if (B!=0)   | D                                    |
| WS, WSRC, WOS    | x                   | B                      | all ls                  | if(B!=1s)   | D                                    |
| W1C, W1CRS       | A                   | B                      | ~A & B                  | if(B!=D)    | ~D                                   |
| W1S, W1SRC       | A                   | B                      | A \| B                  | if(B!=D)    | D                                    |
| W1T              | A                   | B                      | A ^ B                   | if(B!=D)    | D ^ B                                |
| W0C, W0CRS       | A                   | B                      | A & B                   | if(B!=D)    | D                                    |
| W0S, W0SRC       | A                   | B                      | ~A \| B                 | if(B!=D)    | ~D                                   |
| W0T              | A                   | B                      | ~A ^ B                  | if(B!=D)    | ~(D ^ B)                             |
| W1, WO1          | A                   | B                      | first ? A : B Note      | if(B!=D)    | D                                    |
Потому как установки комулятивны до операции обновления, столбец отражаемых значений представляет предыдущее вычисленное желаемое значение из последней операции `set`. Для W1 и WO1 режимов, переменная 'first' в столбце желаемых значений имеет значение единицы до момента записи первого предсказания после аппаратного сброса.

Примеры:
Метод `get()` возвращает вычисленное желаемое значение регистра или поля
```systemverilog
//
// get function prototype
//
function uvm_reg_data_t get(string fname = "",
							int lineno = 0);

//
// Examples - from within a sequence
//
uvm_reg_data_t ctrl_value;
uvm_reg_data_t char_len_value;

// Register level get:
ctrl_value = spi_rm.ctrl.get();

// Field level get (char_len is a field within the ctrl reg):
char_len_value = spi_rm.ctrl.char_len.get();
```

Метод `set()` используется для предустановки желаемого значения регистра или поля для записи в устройство используя метод `update()`.
```systemverilog
//
// set function prototype
//
function void set(uvm_reg_data_t value,
				  string fname = "",
				  int lineno = 0);

// 
// Examples - from within a sequence
//
uvm_reg_data_t ctrl_value;
uvm_reg_data_t char_len_value;

// Register level set:
spi_rm.ctrl.set(ctrl_value);

// Field level set (char_len is a field within the ctrl reg):
spi_rm.ctrl.char_len.set(ctrl_value);
```

#### `get_mirrored_value`
#### `update`
#### `peek` and `poke`

Методы back door доступа, могут быть использованы к полям и к регистрам. Метод `peek()` напрямую читает состояние аппаратного сигнала, `poke()` принудительно изменяет состояние аппаратного сигнала. В обоих случаях желаемое и отражаемое значения в регистровой модели автоматически обновляются
```systemverilog
//
// peek task prototype
//
task peek(output uvm_status_e status,
		  output uvm_reg_data_t value,
		  input string kind = "",
		  input uvm_sequence_base parent = null,
		  input uvm_object extension = null,
		  input string fname = "",
		  input int lineno = 0);

// 
// poke task prototype
//
task poke(output uvm_status_e status,
		  input uvm_reg_data_t value,
		  input string kind = "",
		  input uvm_sequence_base parent = null,
		  input uvm_object extension = null,
		  input string fname = "",
		  input int lineno = 0);

//
// Examples - from within a sequence
//
uvm_reg_data_t ctrl_value;
uvm_reg_data_t char_len_value;

// Register level peek:
ctrl_value = spi_rm.ctrl.peek(status, ctrl_value, .parent(this));

// Field level peek (char_len is a field within the ctrl reg):
spi_rm.ctrl.char_len.peek(status, char_len_value, .parent(this));

// Register level poke:
spi_rm.ctrl.poke(status, ctrl_value, .parent(this));

// Field level poke:
spi_rm.ctrl.char_len.poke(status, char_len_value, .parent(this));
```

#### `randomize`
#### `mirror`

Метод инициирует аппаратное чтение или быстрый доступ, но не возвращает значение данных из устройства. Frontdoor чтение приводит к тому, что предиктор обновляет отражаемое значение, backdoor доступ автоматически обновляет отражаемое значение. Существует опция для сверки считанного из устройства значения с действительным отраженным значением.

Метод `mirror()` может быть вызван для поля, регистра или блочного уровня. на практике следует его использовать на регистровом или блочном уровне для frontdoor доступа, поскольку доступ на уровне поля может не соответствовать характеристикам протокола целевой шины. Вызов `mirror()` блочного уровня приводит к доступу `read`/`peek` ко всем регистрам внутри блока и любого подблока.
```systemverilog
//
// mirror task prototype:
//
task mirror(output uvm_status_e status,
			input uvm_check_e check = UVN_NO_CHECK,
			input uvm_door_e path = UVM_DEFAULT_DOOR,
			input uvm_sequence_base parent = null,
			input int prior = -1,
			input uvm_object extension = null,
			input string fname = "",
			input int lineno = 0);

//
// Examples:
//
// Check the contents of the ctrl register
spi_rm.ctrl.mirror(status, UVM_CHECK);  

// Mirror the contents of spi_rm block via the backdoor
spi_rm.mirror(status, .path(UVM_BACKDOOR));  
```

#### `reset`
#### `get_reset`

### UVM Register Access Method Arguments

Некоторые методы доступа к регистрам содержать большое количество аргументов. Большинство из них имеют значения по умолчанию, так что пользователь не должен определять каждый из аргументов при вызове метода. Требуемые аргументы более или менее одинаковы для тех вызовов, для которых они требуются и в следующей таблице кратко описывается их назначение:

| Argument  | Type              | Default Value                         | Purpose                                                                                                              |
| --------- | ----------------- | ------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| status    | uvm_status_e      | None, должен быть заполнен аргументом | Для возврата статуса вызова метода - может принять значение UVM_IS_OK, UVM_NOT_OK, UVM_IS_X                          |
| value     | uvm_reg_data_t    | None                                  | Для передачи значения данных, выход при чтении, вход при записи                                                      |
| path      | uvm_door_e        | UVM_DEFAULT_DOOR                      | Для определения используемого способа доступа, может быть UVM_FRONTDOOR, UVM_BACKDOOR, UVM_PREDICT, UVM_DEFAULT_DOOR |
| parent    | uvm_sequence_base | null                                  | Для определения какой сиквенс должен быть использован при взаимодействии с сиквенсером                               |
| map       | uvm_reg_map       | null                                  | Для определения карты регистровой модели используемой для обращений                                                  |
| prior     | int               | -1                                    | Для определения приоритета элемента последовательности в целевом сиквенсоре                                          |
| extension | uvm_object        | null                                  | Позволяет передать объект для расширения вызова                                                                      |
| fname     | string            | ""                                    | Используется для отчетности, чтобы привязать вызов метода к имени файла                                              |
| lineno    | int               | 0                                     | Используется для отчетности, чтобы привязать вызов метода к номеру строки                                            |
| kind      | string            | "HARD"                                | Используется для определения типа сброса                                                                             |

### UVM Register Access Method Summary

В следующей таблице приведены различные методы доступа к регистру и уровень на котором они могут быть использованы для backdoor и frontdoor доступов.

![[Pasted image 20250311185920.png]]


## Memory-Level Stimulus

### Memory Model Overview

Модель регистра UVM также поддерживает доступ к памяти. Области памяти внутри ТУ представляются моделями памяти, у которых есть настроенная ширина а диапазон и которые размещены по сдвигу, определенному в регистровой карте. Модель памяти определена одним из режимов доступа read-write, read-only или write-only.

В отличие от регистровой модели, модели памяти не хранят состояние, а просто предоставляют уровень доступа к памяти. Причина тому то, что хранение содержимого памяти привело бы к серьезным накладным расходам при симуляции, и то, что области аппаратной памяти ТУ уже реализованы с использованием моделей, которые предлагают альтернативные способы верификации. Модель памяти поддерживает и front door и back door доступы.

### Memory Model Access Methods

Модель памяти поддерживает 4 типа методов доступа:
- `read`
- `write`
- `burst_read`
- `burst_write`

#### Memory read

Метод `read()` используется для чтения из ячейки памяти, адрес ячейки - сдвиг внутри области памяти, а не абсолютный адрес памяти. Это позволяет перемещать стимулы, обращаемые к памяти, а следовательно переиспользовать их.

```systemverilog
//
// memory read method prototype
//
task uvm_mem::read(output uvm_status_e status,  // outcome of the write cycle
				   input uvm_reg_addr_t offset,  // offset address within the mem
				   output uvm_reg_data_t value,  // read data
				   input uvm_door_e path = UVM_DEFAULT_DOOR,  // front or backdoor
															 // access
				   input uvm_reg_map map = null,  // which map
				   input uvm_sequence_base parent = null,  // parent sequence
				   input int prior = -1,  // priority on the target sequencer
				   input uvm_object extension = null,  // object allowing method
				   input string fname = "",  // filename for messaging
				   input int lineno = 0);  // file line number for messaging

// 
// examples:
//
mem_ss.mem_1.read(status, 32'h1000, read_data, .parent(this));  // default map
mem_ss.mem_1.read(status, 32'h2000, read_data, .parent(this), .map(AHB_2_map));
```

#### Memory write

Метод `write()` используется для записи в ячейку памяти, и так же как и метод `read()` адрес записываемой ячейки - сдвиг внутри области памяти.

```systemverilog
//
// memory write method prototype
//
task uvm_mem::write(output uvm_status_e status,
					input uvm_reg_addr_t offset,
					input uvm_reg_data_t value,
					input uvm_door_e path = UVM_DEFAULT_DOOR,
					input uvm_reg_map map = null,
					input uvm_sequence_base parent = null,
					input int prior = -1,
					input uvm_object extension = null,
					input string fname = "",
					input int lineno = 0);
```

## Register Sequence Examples


## Register-Level Functional Coverage

### Register Based Functional Coverage Overview

UVM поддерживает три способа сбора функционального покрытия, основанных на состоянии регистра:
- Автоматический сбор покрытия регистра при каждом доступе, основанный на группах покрытия внутри регистровой модели
- Управляемый сбор покрытия регистра, основанный на группах покрытия внутри модели регистра, с помощью вызова метода снаружи модели
- По ссылки из внешней группы покрытия, которая сэмплирует значение регистра через указатель регистровой модели

Большинство генераторов регистровой модели позволяют пользователям специфицировать автоматическую генерацию групп, основанную на бите поля или содержимом регистра. Ничего, если у вас узкий бит поля и вы интересуетесь всеми состояниями, которые поле может принять, но они быстро теряют смысл и просто перегружают симуляцию при минимальной отдаче. Для того, чтобы собрать значимое функциональное покрытие регистра, нужно определить покрытие в терминах пересечений значений (`cross`) нескольких регистров и, возможно, нерегистровых сигналов и/или переменных. Ваша генерация регистровой модели может помочь поддержать этот уровень сложности, но, если нет, довольно просто реализовать внешний компонент сбора функционального покрытия, ссылающегося на регистровую модель.

Рекомендуемый подход - использовать внешние группы покрытия, которые сэмплируют значения регистров через указатель регистровой модели.

### Controlling Register Model Functional Coverage Collection

Регистровая модель может содержать множество групп покрытия, что может потенциально оказать серьезное влияние на производительность симуляции. Поэтому есть различные внутренние блокировки, встроенные в регистровую модель, позволяющие вам определять какой тип модели покрытия вы хотите использовать, а также включать или выключать сбор покрытия во время выполнения тестового сценария. Перечисляемый тип с побитовым отображением используется чтобы включать различные модели покрытия, доступные значения перечисления:

| enum value           | Coverage Enabled                                                      |
| -------------------- | --------------------------------------------------------------------- |
| `UVM_NO_COVERAGE`    | Все покрытия выключены                                                |
| `UVM_CVR_REG_BITS`   | Сбор покрытие для битов, записанных или считанных из регистров        |
| `UVM_CVR_ADDR_MAP`   | Сбор покрытия для записываемых или считываемых адресов адресной карты |
| `UVM_CVR_FIELD_VALS` | Сбор покрытия для значений, хранимых в полях регистров                |
| `UVM_CVR_ALL`        | Сбор всех покрытий                                                    |
Перечисление с битовым отображением позволяет включить несколько моделей покрытия одним присваиванием логического ИЛИ нескольких разных значений, например `set_coverage(UVM_CVR_ADDR_MAP + UVM_CVR_FIELD_VALS)`

Модель регистра может содержать группы покрытия, которые были назначены каждой из активных категорий, а в целом покрытие модели регистра устанавливается статическим методом класса `uvm_reg` - `include_coverage()`. Этот метод должен быть вызван до того, как собрана модель регистра, поскольку он создает входы в базу данных ресурсов, на которые ориентируется регистровая модель при выполнении своего метода `build()`, чтобы определить какие группы покрытий должны быть созданы.
```systemverilog
//
// From the SPI test base
//
// Build the env, create the env configuration
// including any sub configurations and assigning virtual interfaces
function void spi_text_base::build_phase(uvm_phase build);
	// env configuration
	m_env_cfg = spi_env_config::type_id::create("m_env_cfg");
	// Register model
	// Enable all types of coverage available in the register model
	uvm_reg::include_coverage("*", UVM_CVR_ALL);
	// Create the register mdoel:
	spi_rm = spi_reg_block::type_id::create("spi_rm");
	// Build and configure the register model
	spi_rm.build();
```

По мере построения модели регистра включается сбор покрытия для различных категорий, которые были включены. Сбор покрытия для категории групп внутри иерархического объекта регистровой модели может управляться методом `set_coverage()` в конъюнкции с методами `has_coverage()` и `get_coverage()`.

### Register Model Coverage Sampling

Группы покрытия внутри регистровой модели в большинстве случаев будут определены моделью спецификации и процессом генерации и конечный пользователь может не знать как они реализованы. Группы покрытия внутри регистровой модели могут сэмплироваться одним из двух способов.

Некоторые группы покрытия в регистровой модели сэмплируютcя как побочный эффект доступа к регистру, т. е. автоматически. Для каждого доступа к регистру происходит автоматический сбор покрытия в регистре и в блоке, который его содержит. Этот тип покрытия важен для получения данных покрытия статистики доступа к регистру и информации, которая может быть связана с доступом к конкретному регистру.

Другие группы покрытия регистровой модели сэмплируются только когда тестовое окружение вызывает метод `sample_values()` из компонента или сиквенса где-либо в тестбенче. Это позволяет собирать более специализированное покрытие. Потенциальные области применения:
- Сэмплирование состояния регистра (конфигурации ТУ), когда происходит специфичное событие, как, например, прерывание
- Сэмплирование состояния регистра при записи определенного регистра

### Referencing The Register Model Data In External Functional Coverage Monitors (Recommended)

Альтернативный способ реализации функционального покрытия, основанного на регистрах - создать компонент монитор для функционального покрытия отдельно от регистровой модели, но сэмплировать значения внутри регистровой модели. Преимущества такого подхода:
- Группы покрытия внутри внешнего монитора могут быть разработаны отдельно от реализации регистровой модели
- Сэмплирование групп покрытия может легче управляться 
- Возможно смешивать, пересекать сэмплированные значения регистровой модели с сэмплированными значениями других переменных тестового окружения

Следующий пример показывает монитор функционального покрытия из тестового окружения SPI, который ссылается на регистровую модель SPI
```systemverilog
class spi_reg_functional_coverage extends uvm_subscriber #(apb_seq_item);
	`uvm_component_utils(spi_reg_functional_coverage)

	logic[4:0] address;
	bit wnr;
	spi_reg_block spi_rm;

	// Checks that the SPI master registers have
	// all been accessed for both reads and writes
	covergroup reg_rw_cov;
		option.per_instance = 1;
		ADDR: coverpoint address {
			bins DATA0 = {0};
			bins DATA1 = {4};
			bins DATA2 = {8};
			bins CTRL = {5'h10};
			bins DIVIDER = {5'h14};
			bins SS = {5'h18};
		}
		CMD: coverpoint wnr {
			bins RD = {0};
			bins WR = {1};
		}
		RW_CROSS: cross CMD, ADDR;
	endgroup: reg_rw_cov

	//
	// Checks that we have tested all possible modes of operation
	// for the SPI master
	//
	// Note that the field value is 64 bits wide, so only the relevant
	// bits are used
	covergroup combination_cov;
		option.per_instance = 1;
		ACS: coverpoint spi_rm.ctrl_reg.acs.value[0];
		IE: coverpoint smi_rm.ctrl_reg.ie.value[0];
		LSB: coverpoint spi_rm.ctrl_reg.lsb.value[0];
		TX_NEG: coverpoint spi_rm.ctrl_reg.tx_neg.value[0];
		RX_NEG: coverpoint spi_rm.ctrl_reg.rx_neg.value[0];
		// Suspect character length - there may be more
		CHAR_LEN: coverpoint spi_rm.ctrl_reg.char_len.value[6:0] {
			bins LENGTH[] = {0, 1, [31:33], [63:65], [95:97], 126, 127};
		}
		CLK_DIV: coverpoint spi_rm.divider_reg.ratio.value[7:0] {
			bins RATIO[] = {16'h0, 16'h1, 16'h2, 16'h4, 16'h8, 
							16'h10, 16'h20, 16'h40, 16'h80};
		}
		COMB_CROSS: cross ACS, IE, LSB, TX_NEG, RX_NEG, CHAR_LEN, CLK_DIV;
	endgroup: combination_cov

	extern function new(strin name = "spi_reg_functional_coverage",
						uvm_component_parent = null);
	extern function void write(T t);

endclass: spi_reg_functional_coverage

function spi_reg_funcional_coverage::new(string name = "spi_reg_funcional_coverage",
										uvm_component_parent = null);
	super.new(name, parent);
	reg_rw_cov = new();
	combination_cov = new();
endfunction: new

function void spi_reg_funcional_coverage::write(T t);
	// Register coverage first
	address = t.addr[4:0];
	wnr = t.we;
	reg_rw_cov.sample();
	// Sample the combination covergroup when go_bsy is true
	if (address == 5'h10) begin
		if (wnr) begin
			if (t.data[8] == 1) begin
				combination_cov.sample();  // TX started
			end
		end
	end
endfunction: write
```

### Coding Guideline: Wrap covergroups within uvm_objects

Группу покрытия стоит реализовывать внутри класса-обёртки, унаследованного от `uvm_object`.

**Обоснование**:

У обёртывания группы покрытия таким способом следующие преимущества:
- `uvm_object` может быть сконструирован в любое время - так и группа покрытия может создана в любое время, что поддерживает условное отложенное конструирование.
- Класс-обёртка группы покрытия может быть переписан из фабрики, что позволяет заместить группу покрытия на альтернативную, если потребуется.
- Это преимущество может стать более актуальным при использовании различных активных фаз в будущем.

**Example:**

```systemverilog
class covergroup_wrapper extends uvm_object;
	`uvm_object_utils(covergroup_wrapper)

	covergroup cg(string name) with function sample(my_reg reg, bin is_read);
		option.name = name;
		CHAR_LEN: coverpoint reg.char_len {
			bins len_5 = {2'b00};
			bins len_6 = {2'b01};
			bins len_7 = {2'b10};
			bins len_8 = {2'b11};
		}
		PARITY: coverpoint reg.parity {
			bins parity_on = {1'b1};
			bins parity_off = {1'b0};
		}
		ALL_OPTIONS: cross CHAR_LEN, PARITY;
	endgroup: cg

	function new(string name = "covergroup_wrapper");
		super.new(name);
		cg = new();
	endfunction

	function voud sample(my_reg reg_in, bit is_read_in);
		cg.sample(reg_in, is_read_in);
	endfunction: sample

endclass: covergroup_wrapper
```