<!--
SPDX-FileCopyrightText: © 2023 Jinwoo Park (pmnxis@gmail.com)

SPDX-License-Identifier: MIT OR Apache-2.0
-->

# `novella`

EEPROM management layer written in Rust, considering wear-leveling, checksum, and page-size

## Example usage
```
Memory Map - Assume 2KB (16KBits) EEPROM.
+----------- 2Kbyte EEPROM (Lower) -----------+-------------- 2Kbyte EEPROM (Higher) --------------+
|                                                                                                  |
|   Section 0 (0x00-0xFF bytes)   p1_card_cnt    Section 4 (0x400-0x47F bytes) hw_boot_cnt         |
|   +-----------------------------------------+  +-----------------------------------------+       |
|   | Page 0  | last_time | p1_card_cnt | CRC |  | Page 0  | last_time | hw_boot_cnt | CRC |       |
|   | Page 1  | last_time | p1_card_cnt | CRC |  | ...     | ...       | ...         | ... |       |
|   | ...     | ...       | ...         | ... |  | Page 7  | last_time | hw_boot_cnt | CRC |       |
|   | Page 15 | last_time | p1_card_cnt | CRC |  +-----------------------------------------+       |
|   +-----------------------------------------+                                                    |
|                                                                                                  |
|   Section 1 (0x100-0x1FF bytes) p2_card_cnt    Section 5 (0x47F-0x4FF bytes) terminal_id[0..6]   |
|   +-----------------------------------------+  +-----------------------------------------------+ |
|   | Page 0  | last_time | p2_card_cnt | CRC |  | Page 0  | last_time | terminal_id[0..6] | CRC | |
|   | Page 1  | last_time | p2_card_cnt | CRC |  | ...     | ...       | ...               | ... | |
|   | ...     | ...       | ...         | ... |  | Page 7  | last_time | terminal_id[0..6] | CRC | |
|   | Page 15 | last_time | p2_card_cnt | CRC |  +-----------------------------------------------+ |
|   +-----------------------------------------+                                                    |
|                                                Section 0..=3 (big sections, 16 page-ish data)    |
|   Section 2 (0x200-0x2FF bytes) p1_coin_cnt    Section  0 : p1_card_cnt    u32     4 bytes       |
|   +-----------------------------------------+  Section  1 : p1_card_cnt    u32     4 bytes       |
|   | Page 0  | last_time | p1_coin_cnt | CRC |  Section  2 : p1_coin_cnt    u32     4 bytes       |
|   | Page 1  | last_time | p1_coin_cnt | CRC |  Section  3 : p2_coin_cnt    u32     4 bytes       |
|   | ...     | ...       | ...         | ... |                                                    |
|   | Page 15 | last_time | p1_coin_cnt | CRC |  Section 4..=11 (small sections, 8 page-ish data)  |
|   +-----------------------------------------+  Section  4 : hw_boot_cnt    u32     4 bytes       |
|                                                Section  5 : terminal_id[0..=5]     6 bytes       |
|   Section 3 (0x300-0x3FF bytes) p2_coin_cnt    Section  6 : terminal_id[6..=9]     4 bytes       |
|   +-----------------------------------------+  Section  7 : terminal_id_ext[0..=2] 3 bytes       |
|   | Page 0  | last_time | p2_coin_cnt | CRC |  Section  8 : card_port1_backup      6 bytes       |
|   | Page 1  | last_time | p2_coin_cnt | CRC |  Section  9 : card_port2_backup      6 bytes       |
|   | ...     | ...       | ...         | ... |  Section 10 : card_port3_backup      6 bytes       |
|   | Page 15 | last_time | p2_coin_cnt | CRC |  Section 11 : card_port4_backup      6 bytes       |
|   +-----------------------------------------+                                                    |
+--------------------------------------------------------------------------------------------------+

  Single Page Structure, M24C16's single page size is 16 bytes.
  Write cycle endurance of each page is 1,200,000 ~ 4,000,0000
  +-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+
  | 0x0 | 0x1 | 0x2 | 0x3 | 0x4 | 0x5 | 0x6 | 0x7 | 0x8 | 0x9 | 0xA | 0xB | 0xC | 0xD | 0xE | 0xF |
  +-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+
  | last_time : embassy_time::Instant (inner:u64) |   Actual Data (Max 6 byte-size)   |   CRC16   |
  +-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+
```

## License
This program and the accompanying materials are made available under the terms
of the Apache Software License 2.0 which is available at
https://www.apache.org/licenses/LICENSE-2.0, or the MIT license which is 
available at https://opensource.org/licenses/MIT
