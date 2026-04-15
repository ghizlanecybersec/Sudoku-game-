import random
import copy

def is_valid(board, row, col, num):
    for x in range(9):
        if board[row][x] == num: return False
    for x in range(9):
        if board[x][col] == num: return False
    start_row, start_col = 3 * (row // 3), 3 * (col // 3)
    for i in range(3):
        for j in range(3):
            if board[start_row + i][start_col + j] == num:
                return False
    return True

def solve_sudoku(board):
    for i in range(9):
        for j in range(9):
            if board[i][j] == 0:
                numbers = list(range(1, 10))
                random.shuffle(numbers)
                for num in numbers:
                    if is_valid(board, i, j, num):
                        board[i][j] = num
                        if solve_sudoku(board):
                            return True
                        board[i][j] = 0
                return False
    return True

def generate_sudoku(difficulty):
    # 1. عمّر لوحة كاملة = الحل الصحيح
    full_board = [[0 for _ in range(9)] for _ in range(9)]
    solve_sudoku(full_board)

    # 2. نسخ منها اللوحة ديال اللاعب
    puzzle = copy.deepcopy(full_board)

    # 3. حيّد الأرقام حسب الصعوبة
    cells = [(i, j) for i in range(9) for j in range(9)]
    random.shuffle(cells)
    cells_to_remove = int(81 * difficulty)

    for i, j in cells[:cells_to_remove]:
        puzzle[i][j] = 0

    return puzzle, full_board # نرجعو اللغز + الحل

def print_board(board):
    for i in range(9):
        if i % 3 == 0 and i!= 0:
            print("- - - - - - -")
        for j in range(9):
            if j % 3 == 0 and j!= 0:
                print("| ", end="")
            if board[i][j] == 0:
                print("_", end=" ")
            else:
                print(board[i][j], end=" ")
        print()

def check_solution(player_board, correct_board):
    return player_board == correct_board

def get_difficulty():
    print("حدد المستوى:")
    print("1. سهل (35 خانة محذوفة)")
    print("2. متوسط (45 خانة محذوفة)")
    print("3. صعب (55 خانة محذوفة)")
    while True:
        choice = input("اختر رقم: ")
        if choice == "1": return 35/81
        elif choice == "2": return 45/81
        elif choice == "3": return 55/81
        else: print("إدخال غير صالح. حاول مرة أخرى.")

def hint(puzzle, player_board, correct_board):
    empty_cells = [(i,j) for i in range(9) for j in range(9) if player_board[i][j] == 0]
    if empty_cells:
        i, j = random.choice(empty_cells)
        player_board[i][j] = correct_board[i][j]
        print(f"تلميح: ({i+1},{j+1}) = {correct_board[i][j]}")
    else:
        print("اللوحة عامرة.")
    return player_board

def undo(player_board, history):
    if history:
        return history.pop()
    else:
        print("لا توجد خطوات للتراجع.")
        return player_board

def play_game():
    difficulty = get_difficulty()
    puzzle, correct_board = generate_sudoku(difficulty)
    player_board = copy.deepcopy(puzzle) # هادي لي غيلعب بها
    history = []

    print("\nلوحة Sudoku:")
    print_board(player_board)

    while True:
        print("\nالأدوات:")
        print("1. إدخال رقم")
        print("2. تلميح")
        print("3. تراجع")
        print("4. تحقق من الحل")
        print("5. استسلام")

        choice = input("اختر رقم: ")

        if choice == "1":
            try:
                i = int(input("ادخل رقم الصف (1-9): ")) - 1
                j = int(input("ادخل رقم العمود (1-9): ")) - 1
                if 0 <= i < 9 and 0 <= j < 9:
                    if puzzle[i][j] == 0: # غير الخانات لي كانت خاوية من الأول
                        history.append(copy.deepcopy(player_board))
                        num = int(input(f"ادخل الرقم في ({i+1},{j+1}) (1-9): "))
                        if 1 <= num <= 9:
                            player_board[i][j] = num
                        else:
                            print("الرقم يجب أن يكون بين 1 و 9.")
                            history.pop()
                    else:
                        print("لا يمكن تغيير هذه الخانة. كانت موجودة من الأول.")
                else:
                    print("الصف والعمود خاصهم يكونو بين 1 و 9.")
            except ValueError:
                print("دخل غير أرقام الله يخليك.")
            print_board(player_board)

        elif choice == "2":
            history.append(copy.deepcopy(player_board))
            player_board = hint(puzzle, player_board, correct_board)
            print_board(player_board)

        elif choice == "3":
            player_board = undo(player_board, history)
            print_board(player_board)

        elif choice == "4":
            if check_solution(player_board, correct_board):
                print("✅ مبروك! الحل صحيح 100%")
                break
            else:
                print("❌ مازال كاين أخطاء. كمّل المحاولة")

        elif choice == "5":
            print("الحل الصحيح كان:")
            print_board(correct_board)
            break
        else:
            print("إدخال غير صالح.")

    # إعادة اللعب
    if input("\nبغيتي تعاود تلعب؟ نعم/لا: ").lower() == "نعم":
        play_game()
    else:
        print("إلى اللقاء!")

play_game()
