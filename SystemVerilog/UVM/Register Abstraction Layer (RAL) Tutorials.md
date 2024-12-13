
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
| `get_reg_by_offset` | Получить отображение регистра по смещению                  |
| `get_mem_by_offset` | Получить отображение памяти по смещению                    |

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

В тестовом окружении конструируется объект модели регистра, а его дескриптор должен быть передан по окружению используя конфигурацию или resource mechanism.
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

Нижний слой поля, соответствующий одному или более битам внутри регистра. Каждое поле - создание экземпляра класса `uvm_reg_field`. Поля содержатся внутри класса `uvm_reg` и они конструируются и конфигурируются методом `configure()`:

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

**Registers**

Регистры моделируются расширением класса `uvm_reg` , с наполнением его объектами `field`. В целом, характеристики регистра определяются его конструктором:

```SystemVerilog
// uvm_reg constructor prototype:
function new (
	string name="",  // register name
	int unsigned n_bits,  // register width in bits
	int has_coverage  // coverage model supported by the register
);
```

Класс `register` содержит `build` метод, используемый для создания и конфигурации полей. Следует отметить, что этот метод не вызывается в UVM build фазе, поскольку регистр - это `uvm_object`, а не компонента

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
	uvm_reg_field reserved;
	rand uvm_reg_field char_len;
	
	function new(string name = "ctrl");
		super.new(name, 14, UVM_NO_COVERAGE);
		
	endfunction

	virtual function void build();
		acs = uvm_reg_field::type_id::create("acs");
		ie = uvm_reg_field::type_id::create("ie");
		...
		char_len = uvm_reg_field::type_id::create("char_len");

		acs.configure(this, 1, 13, "RW", 0, 1'b0, 1, 1, 0);
		ie.configure(this, 1, 11, "RW", 0, 1'b0, 1, 1, 0);
		...
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
class mem_1_model extends uvm_mem;
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
function uvm_reg_map create_map(string name,  // name of map
							    uvm_reg_addr_t base_addr,  // maps base addr
							    int unsigned n_bytes,  // map access width (bytes)
							    uvm_endianness_e endian,  // endianess of map
							    bit byte_addressing = 1);  // whether byte addr
														   // is supported

// example:
AHB_map = create_map("AHB_map", 'h0, 4, UVM_LITTLE_ENDIATN);
```

Параметр `n_bytes` - ширина слова шины, к которой привязана карта. Если ширина регистра превышает ширину шины, больше чем одно обращение к шине необходимо, чтобы записать или прочитать регистр через данную шину. Аргумент `byte_addressing` влияет на то, как адрес увеличивается при последовательных обращениях.  Например, если `n_bytes=4` и `byte_addressing=0`, тогда доступ к регистру шириной 64 биты и смещением 0 произойдет за два обращения к адресам 0 и 1. С `byte_addressing=1` такой же запрос будет результатом двух доступов к адресам 0 и 4.
Значение по умолчанию для `byte_addressing` - 1.

Первая карта, созданная внутри блока регистра будет назначена картой по умолчанию.

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

	uvm_reg_map APB_map;

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

Финальное выражение в `build` методе - метод `lock_model()`, используется чтобы финализировать отображение адресов и гарантировать то, что модель не сможет быть изменена другим пользователем.

**Hierarchical Register Block**

Регистровый блок из предыдущего примера может быть использован блочной верификации, но если SPI интегрирован в больший дизайн, регистровый блок SPI может быть комбинирован с другими регистровыми блока в другом блоке, чтобы создать регистровую модель. Группа блоков включает каждый подблок и добавляет их в новую адресную карту кластерного уровня. Этот процесс может быть замещен полной регистровой картой СнК, содержащей несколько вложенных уровней регистровых блоков. Следующий пример кода показывает как это должно быть сделано для подсистемы с SPI мастером и несколькими другими периферийными блоками.

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

Для данного функционального блока может быть множество регистров, каждый из которых имеет адрес, являющийся смещением от базового адреса блока. Для доступа к регистру, процессор будет читать или записывать по этому адресу. ПО инженер использует регистровую карту, чтобы определить как запрограммировать аппаратное устройство, для этого можно воспользоваться набором определенных утверждений, которые отображают имена регистров и полей в численные значения в заголовочном файле, чтобы он мог работать на более абстрактном уровне детализации. К примеру, если он хочет включить прерывания в SPI мастер, он может прочитать из регистра `CTRL`, затем, логически сложить значение с `IE`, а после записать результирующее значение обратно в `CTRL`:

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

Эти поля должны быть отражены к элементам последовательностей целевой шины, что делается путем расширения `uvm_reg_adapter` класса, который содержит два метода: `reg2bus()` и `bus2reg()`, которые должны быть перекрыты. Класс адаптер также содержит два бита характеристик - `supports_byte_enable` и `provides_responses`, они должны быть установлены в соответствии с функциональностью, поддерживаемой целевой шиной и агентом целевой шины.

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

	virtual function uvm_sequence_item reg2bus(
		const ref uvm_reg_bus_op rw
	);
		apb_seq_item apb = apb_seq_item::type_id::create("apb");
		apb.we = (rw.kind == UVM_READ) ? 0 : 1;
		apb.addr = rw.addr;
		apb.data = rw.data;
		return apb;
		
	endfunction: reg2bus

	virtual function void bus2reg(
		uvm_sequence_item bus_item,
		ref uvm_reg_bus_op rw
	);
		apb_seq_item apb;
		if (!$cast(apb, bus_item)) begin
			`uvm_fatal(...)
			return;
		end
		rw.kind = apb_we ? UVM_WRITE : UVM_READ;
		rw.addr = apb.addr;
		rw.data = apb.data;
		rw.status = UVM_IS_OK;
		
	endfunction: bus2reg
endclass: reg2apb_adapter
```

Код адаптера может быть найден в файле reg2apb_adapter.svh к директории /agents/apb_agent/

#### Burst Support

Методы доступа регистровой модели поддерживают единичный доступ к регистрам, и это соответствует обычной модели использования доступа к регистрам - доступ осуществляется индивидуально, а не как часть пакета. Если у вас есть регистры, который необходимо проверить для режима пакетного доступа.

Если вы используете режим пакетного доступа к регистрам, реализация предсказателя должна быть способная идентифицировать пакеты и преобразовывать каждый такт в вызов `predict()` для соответствующего регистра.

#### Common Adapter Issues

Адаптер - важное звено в цепи последовательностной коммуникации между методами доступа регистровой модели и драйвером. Ожидается, что API драйвера-секвенсора выполнено чисто и следует одной из стандартных моделей на стороне драйвера. Бит `provides_responses` должен быть установлен в 1, если ваш драйвер использует `put(rsp)`, чтобы вернуть ответ, и 0, если нет.

Если настройка `provides_responses` будет некорректной, случится одно из двух - либо генерация стимулов заблокируется, либо будет получен мгновенный ответ от `frontdoor` методов доступа с неверными ответами, а активность на шине произойдет позже.

Блокировка происходит, если вы используете модель завершения `get_next_item()`/`item_done()` и бит `provides_responses` установлен в 1 - причина тому то, что адаптер ждет ответа, который никогда не придет.

Мгновенный возврат случается при использовании модель завершения `get()`/`push()` и бит `provides_responses` установлен в 0, а вызов `get()` в коде драйвера моментально разблокирует метод завершения доступа секвенсора. Драйвер в таком случае продолжает выполнять `bus_access` перед тем, как вернуть ответ через метод `put(rsp)`. Адаптер игнорирует ответ. 

Вы также должны убедиться, что оба обращения (записи и чтения) следуют одной и той же модели завершения. В частности, убедитесь, что вы возвращаете ответ из драйвера для запроса записи, когда используете `get()/put(rsp)` модель завершения.

#### Context for the Bus

Если протокол шины, на который направлен адаптер, нуждается в большем количестве информации для правильной работу, существует метод для предоставления контекстной информации адаптеру.


# Integrating a UVM Register Model in a TestBench - Overview

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
	m_env_cfg = spi_enf_config::type_id::create(...);
	
	// register model
	// enable all types of coverage available in the reg model
	uvm_reg::include_coverate("*", UVM_CVR_ALL);
	
	// create the register model:
	spi_rm = spi_reg_block::type_id::create("spi_rm");
	
	// build and configure the reg model
	spi_rm.build();
	
	// assign a handle to the reg model in the env config
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

Уровень адаптации регистра состоит из двух частей, первая часть реализует наслоение стимулов, основанное на последовательности, а вторая часть реализует основанное на анализе обновление регистровой модели, используя компонент предсказатель.

##### Register Sequence Adaption Layer

Адаптация уровней регистровой последовательности  должна быть выполнена на этапе UVM connect phase, когда регистровая модель и компоненты целевого агента известны и уже собраны.

Распределение регистров по уровням для каждого агента интерфейса целевой шины, поддерживаемого регистровой модель., должно выполняться единожды для каждой карты. В блочной среде это происходит в env, но окружение работает на более высоком уровне интеграции, это отображение следует совершить на верхнем уровне окружения. Для того, чтобы определить на верхнем ли уровне находится конкретный env, код должен проверять родителя данного регистрового блока - null или нет.

В следующем коде из метода env connect уровня блока SPI карта APB из регистрового блока SPI накладывается на секвенсор APB агента, используя класс адаптер `reg2apb`  и метод `set_sequencer()` в объекте APB_map:

```systemverilog
/*
 * From the SPI env
 * Register layering adapter:
 */
reg2apb_adapter reg2apb;

function void spi_env::connect_phase(uvm_phase phase);
	if (m_cfg.m_apb_agent_cfg.active == UVM_ACTIVE) begin
		reg2apb = reg2apb_adapter::type_id::...;
		// register sequencer layering part:
		//
		// only set up register sequencer layering
		// if the top level env
		if (m_cfg.spi_rm.get_parent() == null) begin
			m_cfg.spi_rm.APB_map.set_sequencer(
				m_apb_agent.m_sequencer, reg2apb);
		end
	end
	
endfunction: connect_phase
```

##### Register Prediction

По умолчанию регистровая модель использует процесс под названием `explicit_prediction` для обновления данных регистра, основанных на завершении каждой транзакции чтения и записи, которая модель сгенерировала. Явное прогнозирование нуждается в использовании компоненты `uvm_reg_predictor`.

![[Pasted image 20241203164746.png]]

Компонент `uvm_reg_predictor` наследуется от `uvm_subscriber` и параметризуется типом транзакции анализа целевой шины. Он содержит дескрипторы адаптера шины целевого регистра и адресной карты регистровой модели, использованной  для взаимодействия с секвенсором агента шины. Он использует регистровый адаптер, чтобы преобразовывать транзакции анализа из монитора в регистровые транзакции, а затем ищет адрес регистра в соответствующей шине карте регистровой модели и меняет содержимое соответствующего регистра.

Компонент `uvm_reg_predictor` - часть библиотеки UVM и не нуждается в расширении. Однако, для его интегрирования, необходимо позаботиться о следующих вещах:

1. Объявить предиктор, используя sequence_item целевой шины как параметр класса.
2. Создать предиктор в методе `build()` окружения (env)
3. В методе `connect` - установить отображение предиктора на карту регистра целевой регистровой модели
4. В методе `connect` - установить адаптер предиктора как класс адаптер целевого агента 
5. В методе `connect` - соединить экспорт анализа предиктора с портом анализа целевого агента

Предсказатель должен быть включен в каждое место, где есть монитор шины на целевой шине. Необходимый код показан ниже и во второй половине кода метода SPI connect.

```systemverilog
/*
 * From the SPI env
 * 
 * Register predictor:
 */
uvm_reg_predictor #(apb_seq_item) apb2reg_predictor;

function void spi_env::build_phase(uvm_phase phase);
	// ...
```