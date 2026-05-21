
## Navigational example for a 2dgrid


```python
def navigation(width: int, height: int):

    grid = [[0xF]*width for _ in range(height)]

    px, py = 0, 0
    while True:
        clear_screen()
        for y, row in enumerate(grid):
            for x, col in enumerate(row):
                if (x, y) == (px, py):
                    print(" P ", end='')
                else:
                    print(" . ", end='')
            print()
        print(f"\n    pos: ({px}, {py})  |  WASD to move, Q to quit\n")

        choice = input("> ").strip().lower()
        if choice == 'w' and py > 0:
            py -= 1
        elif choice == 's' and py < height - 1:
            py += 1
        elif choice == 'a' and px > 0:
            px -= 1
        elif choice == 'd' and px < width - 1:
            px += 1
        elif choice == 'q':
            break
```