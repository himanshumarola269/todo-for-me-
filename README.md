# todo-for-me-
```python
import tkinter as tk
from tkinter import messagebox
from datetime import datetime


class FuturisticTodo:
    def __init__(self, root):
        self.root = root
        self.root.title("NEXUS // FUTURISTIC TODO")
        self.root.geometry("1000x700")
        self.root.minsize(850, 600)

        self.tasks = []

        # ---------- COLORS ----------
        self.bg = "#070B14"
        self.panel = "#0D1422"
        self.panel2 = "#111B2D"
        self.text = "#EAF2FF"
        self.muted = "#71809A"
        self.neon = "#00F5FF"
        self.green = "#00FF9D"
        self.red = "#FF4D6D"
        self.yellow = "#FFD166"

        self.root.configure(bg=self.bg)

        self.build_ui()
        self.update_clock()
        self.update_stats()

    # ---------- UI ----------
    def build_ui(self):

        # HEADER
        header = tk.Frame(self.root, bg=self.bg)
        header.pack(fill="x", padx=30, pady=(25, 10))

        title = tk.Label(
            header,
            text="NEXUS",
            font=("Helvetica", 30, "bold"),
            fg=self.neon,
            bg=self.bg
        )
        title.pack(side="left")

        subtitle = tk.Label(
            header,
            text="  // FUTURISTIC TASK COMMAND CENTER",
            font=("Helvetica", 11),
            fg=self.muted,
            bg=self.bg
        )
        subtitle.pack(side="left", pady=(12, 0))

        self.clock = tk.Label(
            header,
            font=("Helvetica", 11),
            fg=self.green,
            bg=self.bg
        )
        self.clock.pack(side="right", pady=10)

        # MAIN AREA
        main = tk.Frame(self.root, bg=self.bg)
        main.pack(fill="both", expand=True, padx=30, pady=10)

        # LEFT PANEL
        left = tk.Frame(
            main,
            bg=self.panel,
            highlightbackground="#18263D",
            highlightthickness=1
        )
        left.pack(side="left", fill="y", padx=(0, 15))

        tk.Label(
            left,
            text="NEW TASK",
            font=("Helvetica", 14, "bold"),
            fg=self.neon,
            bg=self.panel
        ).pack(anchor="w", padx=20, pady=(20, 15))

        self.task_entry = tk.Entry(
            left,
            width=28,
            font=("Helvetica", 12),
            bg=self.panel2,
            fg=self.text,
            insertbackground=self.neon,
            relief="flat"
        )
        self.task_entry.pack(padx=20, pady=5, ipady=10)

        tk.Label(
            left,
            text="PRIORITY",
            font=("Helvetica", 9, "bold"),
            fg=self.muted,
            bg=self.panel
        ).pack(anchor="w", padx=20, pady=(18, 5))

        self.priority = tk.StringVar(value="MEDIUM")

        priority_frame = tk.Frame(left, bg=self.panel)
        priority_frame.pack(padx=20)

        for value in ["LOW", "MEDIUM", "HIGH"]:
            tk.Radiobutton(
                priority_frame,
                text=value,
                variable=self.priority,
                value=value,
                bg=self.panel,
                fg=self.text,
                selectcolor=self.panel2,
                activebackground=self.panel,
                activeforeground=self.neon
            ).pack(anchor="w")

        tk.Label(
            left,
            text="CATEGORY",
            font=("Helvetica", 9, "bold"),
            fg=self.muted,
            bg=self.panel
        ).pack(anchor="w", padx=20, pady=(18, 5))

        self.category = tk.StringVar(value="GENERAL")

        categories = [
            "GENERAL",
            "COLLEGE",
            "PROJECT",
            "PERSONAL",
            "URGENT"
        ]

        category_menu = tk.OptionMenu(
            left,
            self.category,
            *categories
        )
        category_menu.config(
            bg=self.panel2,
            fg=self.text,
            activebackground=self.panel2,
            activeforeground=self.neon,
            highlightthickness=0,
            width=22
        )
        category_menu["menu"].config(
            bg=self.panel2,
            fg=self.text
        )
        category_menu.pack(padx=20, pady=5)

        add_button = tk.Button(
            left,
            text="+  ADD TASK",
            command=self.add_task,
            font=("Helvetica", 11, "bold"),
            bg=self.neon,
            fg="#001014",
            activebackground=self.green,
            relief="flat",
            cursor="hand2"
        )
        add_button.pack(
            padx=20,
            pady=25,
            fill="x",
            ipady=8
        )

        # STATS
        self.stats = tk.Label(
            left,
            text="",
            justify="left",
            font=("Helvetica", 10),
            fg=self.muted,
            bg=self.panel
        )
        self.stats.pack(
            anchor="w",
            padx=20,
            pady=10
        )

        # RIGHT PANEL
        right = tk.Frame(
            main,
            bg=self.panel,
            highlightbackground="#18263D",
            highlightthickness=1
        )
        right.pack(side="right", fill="both", expand=True)

        top = tk.Frame(right, bg=self.panel)
        top.pack(fill="x", padx=20, pady=20)

        tk.Label(
            top,
            text="MISSION QUEUE",
            font=("Helvetica", 16, "bold"),
            fg=self.text,
            bg=self.panel
        ).pack(side="left")

        self.search = tk.Entry(
            top,
            width=22,
            font=("Helvetica", 10),
            bg=self.panel2,
            fg=self.text,
            insertbackground=self.neon,
            relief="flat"
        )
        self.search.pack(side="right", ipady=7)
        self.search.bind("<KeyRelease>", lambda e: self.refresh_tasks())

        # TASK LIST
        self.canvas = tk.Canvas(
            right,
            bg=self.panel,
            highlightthickness=0
        )
        self.canvas.pack(
            side="left",
            fill="both",
            expand=True,
            padx=(15, 0),
            pady=(0, 15)
        )

        scrollbar = tk.Scrollbar(
            right,
            command=self.canvas.yview
        )
        scrollbar.pack(
            side="right",
            fill="y",
            pady=(0, 15),
            padx=(0, 10)
        )

        self.canvas.configure(
            yscrollcommand=scrollbar.set
        )

        self.task_container = tk.Frame(
            self.canvas,
            bg=self.panel
        )

        self.canvas_window = self.canvas.create_window(
            (0, 0),
            window=self.task_container,
            anchor="nw"
        )

        self.task_container.bind(
            "<Configure>",
            lambda e: self.canvas.configure(
                scrollregion=self.canvas.bbox("all")
            )
        )

        self.canvas.bind(
            "<Configure>",
            self.resize_canvas
        )

    # ---------- CLOCK ----------
    def update_clock(self):
        now = datetime.now().strftime("%d %b %Y  |  %H:%M:%S")
        self.clock.config(text=now)
        self.root.after(1000, self.update_clock)

    # ---------- ADD TASK ----------
    def add_task(self):

        task_text = self.task_entry.get().strip()

        if not task_text:
            messagebox.showwarning(
                "NEXUS",
                "Enter a task first."
            )
            return

        task = {
            "text": task_text,
            "priority": self.priority.get(),
            "category": self.category.get(),
            "completed": False,
            "created": datetime.now().strftime("%d/%m/%Y %H:%M")
        }

        self.tasks.append(task)

        self.task_entry.delete(0, tk.END)

        self.refresh_tasks()
        self.update_stats()

    # ---------- DELETE ----------
    def delete_task(self, index):
        del self.tasks[index]
        self.refresh_tasks()
        self.update_stats()

    # ---------- COMPLETE ----------
    def toggle_task(self, index):
        self.tasks[index]["completed"] = not self.tasks[index]["completed"]
        self.refresh_tasks()
        self.update_stats()

    # ---------- REFRESH ----------
    def refresh_tasks(self):

        for widget in self.task_container.winfo_children():
            widget.destroy()

        query = self.search.get().lower()

        visible_tasks = [
            (i, task)
            for i, task in enumerate(self.tasks)
            if query in task["text"].lower()
        ]

        if not visible_tasks:
            tk.Label(
                self.task_container,
                text="NO ACTIVE MISSIONS",
                font=("Helvetica", 14, "bold"),
                fg=self.muted,
                bg=self.panel
            ).pack(pady=80)
            return

        for index, task in visible_tasks:

            card = tk.Frame(
                self.task_container,
                bg=self.panel2,
                highlightbackground="#1C304A",
                highlightthickness=1
            )
            card.pack(
                fill="x",
                padx=5,
                pady=7
            )

            # STATUS BUTTON
            status = "✓" if task["completed"] else "○"

            complete_button = tk.Button(
                card,
                text=status,
                command=lambda i=index: self.toggle_task(i),
                font=("Helvetica", 16, "bold"),
                bg=self.panel2,
                fg=self.green if task["completed"] else self.muted,
                activebackground=self.panel2,
                relief="flat"
            )
            complete_button.pack(
                side="left",
                padx=12
            )

            info = tk.Frame(
                card,
                bg=self.panel2
            )
            info.pack(
                side="left",
                fill="both",
                expand=True,
                pady=10
            )

            task_color = self.muted if task["completed"] else self.text

            title = tk.Label(
                info,
                text=task["text"],
                font=("Helvetica", 12, "bold"),
                fg=task_color,
                bg=self.panel2
            )
            title.pack(anchor="w")

            details = tk.Label(
                info,
                text=f"{task['category']}   •   {task['priority']}   •   {task['created']}",
                font=("Helvetica", 8),
                fg=self.muted,
                bg=self.panel2
            )
            details.pack(anchor="w", pady=(3, 0))

            if task["priority"] == "HIGH":
                priority_color = self.red
            elif task["priority"] == "MEDIUM":
                priority_color = self.yellow
            else:
                priority_color = self.green

            indicator = tk.Label(
                card,
                text=f" {task['priority']} ",
                font=("Helvetica", 8, "bold"),
                fg=priority_color,
                bg=self.panel2
            )
            indicator.pack(
                side="right",
                padx=10
            )

            delete_button = tk.Button(
                card,
                text="×",
                command=lambda i=index: self.delete_task(i),
                font=("Helvetica", 15, "bold"),
                fg=self.red,
                bg=self.panel2,
                activebackground=self.panel2,
                relief="flat",
                cursor="hand2"
            )
            delete_button.pack(
                side="right",
                padx=5
            )

    # ---------- STATS ----------
    def update_stats(self):

        total = len(self.tasks)
        completed = sum(
            1 for task in self.tasks
            if task["completed"]
        )

        pending = total - completed

        if total:
            progress = int((completed / total) * 100)
        else:
            progress = 0

        self.stats.config(
            text=(
                f"SYSTEM STATUS\n\n"
                f"TOTAL MISSIONS     {total}\n"
                f"COMPLETED          {completed}\n"
                f"PENDING            {pending}\n\n"
                f"PROGRESS           {progress}%\n\n"
                f"[ {'█' * (progress // 10)}"
                f"{'░' * (10 - progress // 10)} ]"
            )
        )

    # ---------- RESIZE ----------
    def resize_canvas(self, event):
        self.canvas.itemconfig(
            self.canvas_window,
            width=event.width
        )


# ---------- START ----------
if __name__ == "__main__":
    root = tk.Tk()
    app = FuturisticTodo(root)
    root.mainloop()
```
