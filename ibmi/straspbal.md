---
title: IBMi End Allocation Command
description: 
published: true
date: 2022-05-14T15:37:52.981Z
tags: script, ibmi
editor: markdown
dateCreated: 2022-05-14T15:19:27.631Z
---

# Objective
When ending allocation on units in an IBMi ASP, you have to type each unit number in a list.  When dealing with a large number of units that are in a consecutive range, you can use the python script below to create a command to end allocation on IBMi.

# Syntax
```python
from os import system, name

# Clear the command prompt or terminal screen (based on OS type)
if name == 'nt':
    _ = system('cls')
else:
    _ = system('clear')

# Prompt for range start and stop unit numbers
start = input('Enter starting unit number in the range to be removed:  ')
stop = input('Enter ending unit number in the range to be removed:  ')

# Initialize list variable
range_list = []

# Create a list of range values
for i in range(int(start), int(stop) + 1):
    range_list.append(str(i))

# Create command string by using join to add spaces to the values in the list
range_str = "STRASPBAL TYPE(*ENDALC) UNIT(" + ' '.join(range_list) + ')'

# Print the final command to the terminal
print('\n' + range_str + '\n')
```

# Example
```
Enter starting unit number in the range to be removed:  586
Enter ending unit number in the range to be removed:  645

STRASPBAL TYPE(*ENDALC) UNIT(586 587 588 589 590 591 592 593 594 595 596 597 598 599 600 601 602 603 604 605 606 607 608 609 610 611 612 613 614 615 616 617 618 619 620 621 622 623 624 625 626 627 628 629 630 631 632 633 634 635 636 637 638 639 640 641 642 643 644 645)
```