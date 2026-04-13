# verihogg-format_mvp

> Форматтер кода для SystemVerilog.

**verihogg-format** форматирует SystemVerilog стандарта IEEE 1800-2017.

```
verihogg-format src/**/*.sv
```

---

## Установка

Для сборки нужен [Nix](https://nixos.org/download/).

```bash
git clone https://github.com/verihogg/verihogg-format
cd verihogg-format
nix-shell
cmake -B build
cmake --build build
```

Бинарник окажется в `build/bin/verihogg-format`. Добавьте путь в `$PATH` или вызывайте напрямую.

---

## Использование

```bash
# форматировать файлы на месте
verihogg-format mymodule.sv
verihogg-format src/**/*.sv

# проверить без изменений — выйти с кодом 1, если хоть один файл изменился бы (для CI)
verihogg-format --check src/**/*.sv
```

### Коды выхода

| Код | Значение |
|-----|---------|
| `0` | Успех. В режиме `--check`: ни один файл не изменился бы. |
| `1` | В режиме `--check`: один или несколько файлов были бы отформатированы. |
| `2` | Ошибка — не удалось прочитать файл, невалидный конфиг и т.д. |

---

## Конфигурация

Конфигурация не обязательна. Положите `.verihogg.toml` в корень проекта,
чтобы переопределить дефолты. Форматтер ищет его вверх по директориям
от каждого исходного файла.

```toml
indent_width    = 2        # пробелов на уровень отступа
column_limit    = 100      # мягкий предел длины строки
use_tabs        = false

port_alignment  = "align"  # "align" | "flush-left"
endlabel_style  = "always" # "always" | "never" | "preserve"
begin_end_style = "always" # "always" | "preserve"
```

Приоритет:

```
CLI флаги  >  .verihogg.toml  >  встроенные дефолты
```

### Параметры

| Параметр | Дефолт | Описание |
|----------|--------|---------|
| `indent_width` | `2` | Пробелов на каждый уровень отступа. |
| `column_limit` | `100` | Мягкий предел длины строки. Строки, которые нельзя разбить, могут его превысить. |
| `use_tabs` | `false` | Использовать табы вместо пробелов. |
| `port_alignment` | `"align"` | Выравнивать объявления портов по столбцам (`align`) или оставить каждый вплотную к краю (`flush-left`). |
| `endlabel_style` | `"always"` | Добавлять метку после `endmodule` / `endgenerate` (`always`), удалять её (`never`) или оставлять как есть (`preserve`). |
| `begin_end_style` | `"always"` | Всегда оборачивать тела `if`/`else`/`always` из одного выражения в `begin`/`end` (`always`) или оставлять как есть (`preserve`). |

---

## Отключение форматирования

Для ручного выравнивания, которое нужно сохранить
оберните регион в комментарии:

```verilog
// verihogg: off
assign out = (sel == 2'd0) ? a :
             (sel == 2'd1) ? b :
             (sel == 2'd2) ? c : d;
// verihogg: on
```

Всё между маркерами проходит насквозь без изменений.

---

## Пример

Опенсорсный процессор [SCR1](https://github.com/syntacore/scr1) — RISC-V ядро
от Syntacore — форматируется verihogg-format без проблем. Ниже фрагмент
из модуля ALU до и после запуска с дефолтными настройками:

**До**

```verilog
module scr1_pipe_ialu #(parameter SCR1_XLEN = 32)(
input logic clk,input logic rst_n,
input  logic [SCR1_XLEN-1:0]  main_op1,
input logic[SCR1_XLEN-1:0] main_op2,
input  type_scr1_ialu_cmd_e   cmd,
output logic[SCR1_XLEN-1:0] result,
output logic cmp_res
);

always_comb begin
  result='0;
  if(cmd==SCR1_IALU_CMD_AND)
    result=main_op1&main_op2;
  else if(cmd==SCR1_IALU_CMD_OR)
    result=main_op1|main_op2;
end
```

**После**

```verilog
module scr1_pipe_ialu #(
  parameter SCR1_XLEN = 32
) (
  input  logic                   clk,
  input  logic                   rst_n,
  input  logic [SCR1_XLEN-1:0]  main_op1,
  input  logic [SCR1_XLEN-1:0]  main_op2,
  input  type_scr1_ialu_cmd_e    cmd,
  output logic [SCR1_XLEN-1:0]  result,
  output logic                   cmp_res
);

  always_comb begin
    result = '0;
    if (cmd == SCR1_IALU_CMD_AND) begin
      result = main_op1 & main_op2;
    end else if (cmd == SCR1_IALU_CMD_OR) begin
      result = main_op1 | main_op2;
    end
  end
```

## Лицензия

MIT
