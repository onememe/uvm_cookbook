
# UVM Sequence

## Start a sequence

Последовательность запускается вызовом метода `start`, который принимает указатель на секвенсор, через который элементы последовательности будут отправлены на драйвер. Указатель на секвенсор также известен как `m_sequencer`. Метод `start` присваивает указатель на секвенсор в `m_sequencer`, а затем вызывает задачу `body()`.  По завершении задачи `body` взаимодействием с драйвером, метод выполнение возвращается в метод `start()`. Как требует взаимодействие с драйвером, `start` - блокирующий метод.

```systemverilog
// start method definition
virtual task start( 
	uvm_sequencer_base sequencer,
	uvm_sequence_base parent_sequence = null,
	int this_priority = -1;
	bit call_pre_post = 1;
);
```

- `sequencer` - при запуске последовательности необходимо указать, на каком секвенсоре она должна быть запущена, остальные аргументы являются опциональными.
- `parent_sequence` - последовательность, вызывающая текущую. Если родительская последовательность `null`, тогда эта последовательность - корневая, иначе - она предок `parent_sequence`. Методы `pre_do`, `mid_do` и `post_do` родителя будут вызваны при выполнении текущей последовательности.
- `this_priority` - приоритет последовательности (по умолчанию рассматривается приоритет родительской последовательности). Высший приоритет определяется высшим значением.
- `call_pre_post` - по умолчанию равен единице

```systemverilog
assert(child_seq.randomize());
child_seq.start(seqr, parent_seq, priority, call_pre_post);
```

Во время выполнения последовательности через метод `start` вызываются следующие методы:

- `pre_start` - вызывается всегда
- `pre_body`
- `pre_do`
- `mid_do`
- `body`
- `post_do`
- `post_body`
- `post_start` - вызывается всегда

Вызов метода `start` выполняется в следующей последовательности:

```systemverilog
sub_seq.pre_start()
sub_seq.pre_body()          if call_pre_post == 1
  parent_seq.pre_do(0)      if parent_sequence != null
  parent_seq.mid_do(this)   if parent_sequence != null
sub_seq.body                YOUR STIMULUS CODE
  parent_seq.post_do(this)  if parent_sequence != null
sub_seq.post_body()         if call_pre_post == 1
sub_seq.post_start()
```

Чтобы яснее понять, `base_seq` и его потомок `child_seq` приведены в примере ниже, со всеми определенными методами

```systemverilog
class base_seq extends uvm_sequence #(seq_item);
	`uvm_object_utils(base_seq)
	function new(string name = "base_seq");
		super.new(name);
	endfunction
	
	task pre_start();
		`uvm_info(...)
	endtask
	
	task pre_body();
		`uvm_info(...)
	endtask
	
	virtual task pre_do(bit is_item);
		`uvm_info(...)
	
	endtask
	
	virtual function void mid_do(uvm_sequence_item this_item);
		`uvm_info(...)
	endfunction
	
	task body();
		`uvm_info(...)
		`uvm_do(req);
	endtask
	
	virtual function void post_do(uvm_sequence_item this_item);
		`uvm_info(...)
	endfunction
	
	task post_body();
		`uvm_info(...)
	
	endtask
endclass

class child_seq extends base_seq;
	`uvm_object_utils(child_seq)
	function new(string name = "child_seq");
		super.new(name);
	endfunction
```


# Virtual Sequence and Virtual Sequencers

Виртуальная последовательность - ничто иное как контейнер, запускающий множество последовательностей на разных секвенсорах. Виртуальный секвенсор управляет другими секвенсорами, а также не присоединен ни к какому драйверу.

## Virtual Sequence and Virtual Sequencer Usage

В СнК могут быть разные модули, которые взаимодействуют на разных протоколах. Поэтому нам нужны разные драйверы, чтобы подавать сигналы в соответствии с интерфейсами. Так, мы обычно храним раздельные агенты для того, чтобы справляться с разными протоколами. Вследствие этого нам нужно выполнять последовательности на соответствующем секвенсоре.

Другим примером может служить множество ядер в СнК. В СнК может быть представлено несколько ядер, которые могут управляться разными операциями на входе, а также отвечать устройству по разному. В таком случае, опять же, выполнение разных последовательностей на соответствующих секвенсорах становится важным .

Виртуальная последовательность обычно исполняется на виртуальном секвенсоре. Виртуальная последовательность позволяет управлять запуском разных последовательностей.

Рекомендуется использовать виртуальный секвенсор, если вы имеете множество агентов и необходимо скоординировать стимулы.

![[Pasted image 20241205175100.png]]

### Why are the `virtual_sequence` and `virtual_sequencer` named `virtual`?

В SystemVerilog есть виртуальные методы, интерфейсы и классы. Ключевое слово `virtual` - является общим для них. Но, `virtual_sequence` и `virtual_sequencer` не нуждаются в ключевом слове. В UVM нет `uvm_virtual_sequence` и `uvm_virtual_sequencer` базовых классов. Виртуальная последовательность наследуется от `uvm_sequence`. `virtual_sequencer` наследуется от базового класса `uvm_sequencer`.

Виртуальный секвенсор управляет другими секвенсорами. Он не привязан ни к какому драйверу и также не может обрабатывать какие-либо `sequence_items`. Следовательно, его называют `virtual`.


## Examples

### Without virtual sequence and virtual sequencer

```systemverilog
/**
 * No Virtual Sequencer
 */
class core_A_sequencer extends uvm_sequencer #(seq_item);
	`uvm_component_utils(core_A_sequencer)
	function new(string name = "core_A_sequener", uvm_component parent = null);
		super.new(name, parent);
	endfunction
endclass

class core_B_sequencer extends uvm_sequencer #(seq_item);
	`uvm_component_utils(core_B_sequencer)
	function new(string name = "core_B_sequencer", uvm_component parent = null);
		super.new(name, parent);
	endfunction
endclass

// base_test
class base_test extends uvm_test;
	env env_o;
	core_A_seq seqA;
	core_B_seq seqB;
	
	`uvm_component_utils(base_test)
	function new(string name = "base_test", uvm_component parent = null);
		super.new(name, parent);
	endfunction
	
	function void build_phase(uvm_phase phase);
		super.build_phase(phase);
		env_o = env::type_id::create("env_o", this);
	endfunction
	
	task run_phase(uvm_phase phase);
		phase.raise_objection(this);
		seqA = core_A_seq::type_id::create("seqA");
		seqB = core_B_seq::type_id::create("seqB");
		
		seqA.start(env_o.agt_A.seqr_A);
		seqB.start(env_o.agt_B.seqr_B);
		
		phase.drop_objection(this);
	endtask
endclass
```

### With virtual sequence and without a virtual sequencer

```systemverilog
/* Virtual Sequence */
class virtual_seq extends uvm_sequence #(seq_item);
	core_A_seq seqA;
	core_B_seq seqB;
	
	core_A_sequencer seqr_A;
	core_B_sequencer seqr_B;
	
	`uvm_object_utils(virtual_seq)
	function new(string name = "virtual_seq");
		super.new(name);
	endfunction
	
	task body();
		`uvm_info(...);
		seqA = core_A_seq::type_id::create("seqA");
		seqB = core_B_seq::type_id::create("seqB");
	
		seqA.start(seqr_A);
		seqB.start(seqr_B);
	endtask
endclass

/* No Virtual Sequencer */
class core_A_sequencer extends uvm_sequencer #(seq_item);
	`uvm_component_utils(core_A_sequencer)
	function new(string name = "core_A_sequncer", uvm_component parent = null);
		super.new(name, parent);
	endfunction
endclass

class core_B_sequencer extends uvm_sequencer #(seq_item);
	`uvm_component_utils(core_B_sequencer)
	function new(string name = "core_B_sequncer", uvm_component parent = null);
		super.new(name, parent);
	endfunction
endclass
```

### With virtual sequence and virtual sequencer using `p_sequencer` handle

```systemverilog
/* Virtual Sequence */
class virtual_seq extends uvm_sequence #(seq_item);
	core_A_seq seqA;
	core_B_seq seqB;
	
	core_A_sequencer seqr_A;
	core_B_sequencer seqr_B;
	`uvm_object_utils(vitual_seq)
	`uvm_declare_p_sequencer(virtual_sequencer)
	
	function new(string name = "virtual_seq");
		super.new(name);
	endfunction
	
	task body();
		`uvm_info(...)
		seqA = core_A_seq::type_id::create("seqA");
		seqB = core_B_seq::type_id::create("seqB");
		
		seqA.start(p_sequencer.seqr_A);
		seqB.start(p_sequencer.seqr_B);
	endtask
endclass

/* Virtual p_sequencer */
class virtual_sequencer extends uvm_sequencer;
	`uvm_component_utils(virtual_sequencer)
	core_A_sequencer seqr_A;
	core_B_sequencer seqr_B;
	
	function new(string name = "virtual_sequencer", uvm_component parent = null);
		super.new(name, parent);
	endfunction
endclass
```

### With virtual sequence and virtual sequencer but without using `p_sequencer` handle

```systemverilog
// Virtual Sequencer
class virtual_seq extends uvm_sequencer #(seq_item);
	core_A_seq seqA;
	core_B_seq seqB;
	
	core_A_sequencer seqr_A;
	core_B_sequencer seqr_B;
	`uvm_object_utils(vitual_seq)

	function new(string name = "virtual_seq");
		super.new(name);
	endfunction

	task body();
		env env_s;
		`uvm_info(...)
		
		// virtual_sequencer is created in env, so we need env handle to find v_seqr.
		if (!$cast(env_s, uvm_top.find("uvm_test_top.env_o")))
			`uvm_error(...)
		
		seqA.start(env_s.v_seqr.seqr_A);
		seqB.start(env_s.v_seqr.seqr_B);
	endtask
endclass

// Virtual Sequencer
class virtual_sequencer extends uvm_sequencer;
	`uvm_component_utils(virtual_sequencer)
	core_A_sequencer seqr_A;
	core_B_sequencer seqr_B;
	
	function new(string name = "virtual_sequencer", uvm_component parent = null);
		super.new(name, parent);
	endfunction
endclass
```