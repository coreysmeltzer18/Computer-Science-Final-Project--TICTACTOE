import tkinter as tk
from tkinter import messagebox
import random


class TicTacToe:
    def __init__(self, root):
        """Set up everything when the program starts."""
        self.root = root
        self.root.title("Tic Tac Toe")

        # empty 3x3 board
        self.board = [["" for _ in range(3)] for _ in range(3)]
        self.current_player = "X"

        # mode/difficulty stuff
        self.mode = "PVC"
        self.difficulty = "Unbeatable"

        # scores
        self.score_x = 0
        self.score_o = 0

        # move history
        self.history = []

        self.buttons = []
        self.make_gui()
    def make_gui(self):
        """Make all buttons and labels for the game."""
        top = tk.Frame(self.root)
        top.pack(pady=5)

        # switch PvP / PvC
        self.mode_btn = tk.Button(
            top, text="Mode: Player vs Computer", command=self.change_mode
        )
        self.mode_btn.grid(row=0, column=0)

        # pick difficulty
        self.diff_var = tk.StringVar(value="Unbeatable")
        tk.OptionMenu(top, self.diff_var, "Random", "Easy", "Unbeatable",
                      command=self.change_difficulty).grid(row=0, column=1)

        # scoreboard
        self.score_label = tk.Label(self.root, text="Score — X: 0  O: 0")
        self.score_label.pack()

        # board buttons
        board_frame = tk.Frame(self.root)
        board_frame.pack()

        for r in range(3):
            row_list = []
            for c in range(3):
                btn = tk.Button(
                    board_frame, text="", font=("Arial", 24),
                    width=5, height=2,
                    command=lambda rr=r, cc=c: self.player_click(rr, cc)
                )
                btn.grid(row=r, column=c)
                row_list.append(btn)
            self.buttons.append(row_list)

        # move history box
        self.history_box = tk.Listbox(self.root, width=25, height=8)
        self.history_box.pack()

        # reset game button
        tk.Button(self.root, text="New Game", command=self.new_game).pack(pady=5)
    def change_mode(self):
        """Switch between PvP and PvC."""
        if self.mode == "PVC":
            self.mode = "PVP"
            self.mode_btn.config(text="Mode: Player vs Player")
        else:
            self.mode = "PVC"
            self.mode_btn.config(text="Mode: Player vs Computer")
        self.new_game()

    def change_difficulty(self, diff):
        """Set AI difficulty."""
        self.difficulty = diff
    def new_game(self):
        """Reset the board and GUI for a new round."""
        self.board = [["" for _ in range(3)] for _ in range(3)]
        self.current_player = "X"
        self.history = []
        self.history_box.delete(0, tk.END)

        for r in range(3):
            for c in range(3):
                self.buttons[r][c].config(text="", state="normal")
    def player_click(self, r, c):
        """When player clicks a space."""
        if self.board[r][c] != "":
            return  # can't place on filled space

        self.place_move(r, c, self.current_player)

        if self.check_end():
            return

        # AI plays if needed
        if self.mode == "PVC" and self.current_player == "O":
            self.root.after(150, self.ai_move)

    def place_move(self, r, c, player):
        """Put X or O on the board and switch turns."""
        self.board[r][c] = player
        self.buttons[r][c].config(text=player, state="disabled")

        # record the move
        move = f"{player} -> ({r+1},{c+1})"
        self.history.append(move)
        self.history_box.insert(tk.END, move)

        # switch turn
        self.current_player = "O" if player == "X" else "X"

    def ai_move(self):
        """AI picks a move depending on difficulty."""
        if self.difficulty == "Random":
            r, c = self.ai_random()
        elif self.difficulty == "Easy":
            r, c = self.ai_easy()
        else:
            r, c = self.ai_minimax()

        self.place_move(r, c, "O")
        self.check_end()
    # AI difficulty 1: random
    def ai_random(self):
        """AI picks a random empty spot"""
        empty = [(r, c) for r in range(3) for c in range(3)
                 if self.board[r][c] == ""]
        return random.choice(empty)
    # AI difficulty 2: easy
    def ai_easy(self):
        """AI tries to win in one move, otherwise random"""
        # try to win immediately
        for r in range(3):
            for c in range(3):
                if self.board[r][c] == "":
                    self.board[r][c] = "O"
                    if self.check_winner() == "O":
                        self.board[r][c] = ""
                        return (r, c)
                    self.board[r][c] = ""

        # otherwise random
        return self.ai_random()
    # AI difficulty 3: unbeatable
    def ai_minimax(self):
        """Use minimax to find best move."""
        best_score = -999
        best_move = None

        for r in range(3):
            for c in range(3):
                if self.board[r][c] == "":
                    # try move
                    self.board[r][c] = "O"
                    score = self.minimax(False)
                    self.board[r][c] = ""  # undo move

                    if score > best_score:
                        best_score = score
                        best_move = (r, c)

        return best_move
    def minimax(self, is_max_turn):
        """Recursive minimax function."""
        winner = self.check_winner()
        if winner == "O":
            return 1
        if winner == "X":
            return -1
        if self.board_full():
            return 0

        # AI turn (max)
        if is_max_turn:
            best = -999
            for r in range(3):
                for c in range(3):
                    if self.board[r][c] == "":
                        self.board[r][c] = "O"
                        best = max(best, self.minimax(False))
                        self.board[r][c] = ""
            return best

        # player turn (min)
        else:
            best = 999
            for r in range(3):
                for c in range(3):
                    if self.board[r][c] == "":
                        self.board[r][c] = "X"
                        best = min(best, self.minimax(True))
                        self.board[r][c] = ""
            return best
    # WIN CHECKING
    def check_winner(self):
        """Check if anyone has won the game."""
        b = self.board

        # rows + columns
        for i in range(3):
            if b[i][0] == b[i][1] == b[i][2] != "":
                return b[i][0]
            if b[0][i] == b[1][i] == b[2][i] != "":
                return b[0][i]

        # diagonals
        if b[0][0] == b[1][1] == b[2][2] != "":
            return b[0][0]
        if b[0][2] == b[1][1] == b[2][0] != "":
            return b[0][2]

        return None

    def board_full(self):
        """True if no spaces left."""
        return all(self.board[r][c] != "" for r in range(3) for c in range(3))
    
    def check_end(self):
        """Check if game is over (win or draw)."""
        winner = self.check_winner()

        if winner:
            messagebox.showinfo("Game Over", f"{winner} wins!")
            self.update_score(winner)
            self.disable_board()
            return True

        if self.board_full():
            messagebox.showinfo("Game Over", "It's a draw!")
            self.disable_board()
            return True

        return False

    def update_score(self, winner):
        """Update the score label."""
        if winner == "X":
            self.score_x += 1
        else:
            self.score_o += 1
        self.score_label.config(
            text=f"Score — X: {self.score_x}  O: {self.score_o}"
        )

    def disable_board(self):
        """Disable everything when round ends."""
        for r in range(3):
            for c in range(3):
                self.buttons[r][c].config(state="disabled")

# MAIN PROGRAM
if __name__ == "__main__":
    root = tk.Tk()
    TicTacToe(root)
    root.mainloop()

