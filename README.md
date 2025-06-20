# -
Р/З ООП
import tkinter as tk
from random import randint, random


class Bee:
    def __init__(self, id, weight, age):
        self.id = id
        self.weight = weight
        self.age = age
        self.max_age = randint(30, 60)
        self.alive = True

    def die(self):
        self.alive = False


class Queen(Bee):
    def __init__(self, id, weight, age):
        super().__init__(id, weight, age)
        self.productivity = 1.0


class Drone(Bee):
    def __init__(self, id, weight, age):
        super().__init__(id, weight, age)
        self.fertilized = 0


class Worker(Bee):
    def __init__(self, id, weight, age):
        super().__init__(id, weight, age)
        self.job_type = randint(0, 1)


class Larva:
    def __init__(self, id):
        self.id = id
        self.weight = 1
        self.age = 0
        self.development_stage = 0
        self.alive = True

    def die(self):
        self.alive = False


class Hive:
    def __init__(self):
        self.queens = [Queen(1, 10, 0)]
        self.drones = []
        self.workers = [Worker(2, 5, 0), Worker(3, 5, 0)]
        self.larvae = []
        self.dead_bees = []
        self.honey = 1000
        self.day = 0
        self.next_id = 4

    def simulate_day(self):
        self.day += 1

        spawn = min(int(self.honey / 150), 5)
        for _ in range(spawn):
            self.larvae.append(Larva(self.next_id))
            self.next_id += 1

        for larva in self.larvae[:]:
            if larva.alive:
                larva.development_stage += 1
                if larva.development_stage > 5:
                    if random() < 0.8:
                        self.workers.append(Worker(self.next_id, 5, 0))
                    else:
                        self.drones.append(Drone(self.next_id, 5, 0))
                    self.next_id += 1
                    self.larvae.remove(larva)

        for bee in self.queens + self.drones + self.workers:
            consumption = bee.weight * 0.05
            if self.honey >= consumption:
                self.honey -= consumption
            else:
                bee.die()
                self.dead_bees.append(bee)

        for larva in self.larvae[:]:
            if larva.alive:
                consumption = larva.weight * 0.45
                if self.honey >= consumption:
                    self.honey -= consumption
                else:
                    larva.die()
                    self.dead_bees.append(larva)
                    self.larvae.remove(larva)

        for bee in self.workers:
            if bee.job_type == 0:
                self.honey += bee.weight * 0.25

        for corpse in self.dead_bees[:]:
            for worker in self.workers:
                if worker.job_type == 1 and getattr(corpse, "weight", 0) < worker.weight:
                    self.dead_bees.remove(corpse)
                    break


class VisualInterface:
    def __init__(self):
        self.hive = Hive()

        self.window = tk.Tk()
        self.window.title("🐝 Жизнь пчелинного улья 🐝")
        self.window.attributes('-fullscreen', True)
        self.window.configure(bg="#FFF9C4")

        self.screen_width = self.window.winfo_screenwidth()
        self.screen_height = self.window.winfo_screenheight()

        self.canvas = tk.Canvas(self.window, bg="#81C784", width=self.screen_width, height=self.screen_height)
        self.canvas.pack(fill=tk.BOTH, expand=True)

        self.info = tk.Frame(self.window, bg="#FDD835", padx=10, pady=10)
        self.info.place(relwidth=0.3, relheight=1)

        self.label_day = tk.Label(self.info, text="День: 0", bg="#FDD835", font=("Arial", 16, "bold"), fg="black")
        self.label_day.pack(pady=5)

        self.label_queens = tk.Label(self.info, text="Пчел: 1", bg="#FDD835", font=("Arial", 12), fg="black")
        self.label_queens.pack()

        self.label_drones = tk.Label(self.info, text="Трутней: 0", bg="#FDD835", font=("Arial", 12), fg="black")
        self.label_drones.pack()

        self.label_workers = tk.Label(self.info, text="Рабочих пчел: 2", bg="#FDD835", font=("Arial", 12), fg="black")
        self.label_workers.pack()

        self.label_larvae = tk.Label(self.info, text="Личинок: 0", bg="#FDD835", font=("Arial", 12), fg="black")
        self.label_larvae.pack()

        self.label_dead = tk.Label(self.info, text="Мертвых пчел: 0", bg="#FDD835", font=("Arial", 12), fg="black")
        self.label_dead.pack()

        self.label_honey = tk.Label(self.info, text="Запасы меда: 1000", bg="#FDD835", font=("Arial", 12), fg="black")
        self.label_honey.pack()

        self.button_frame = tk.Frame(self.window, bg="#FDD835")
        self.button_frame.place(relx=0.3, rely=0.9, relwidth=0.7, relheight=0.1)

        self.btn_simulate = tk.Button(self.button_frame, text="Симулировать день", command=self.simulate_day, bg="#FF5722", fg="white", font=("Arial", 12))
        self.btn_simulate.pack(side=tk.LEFT, padx=20, expand=True)

        self.btn_exit = tk.Button(self.button_frame, text="Выход", command=self.exit_program, bg="#E91E63", fg="white", font=("Arial", 12))
        self.btn_exit.pack(side=tk.RIGHT, padx=20, expand=True)

        self.bee_objects = []

        self.place_flowers()

        self.hive_center = (self.screen_width // 2, self.screen_height // 2)
        self.hive_radius = 40
        self.draw_hive()

        for bee in self.hive.workers:
            self.create_bee_visual(bee)

        self.animate()

    def place_flowers(self):
        flower_emojis = ["🌸", "🌻", "🌼", "🌷", "🌺"]
        for _ in range(60):
            x = randint(50, self.screen_width - 50)
            y = randint(50, self.screen_height - 50)
            emoji = flower_emojis[randint(0, len(flower_emojis) - 1)]
            self.canvas.create_text(x, y, text=emoji, font=("Arial", 12))

    def create_bee_visual(self, bee):
        x = randint(100, self.screen_width - 100)
        y = randint(100, self.screen_height - 100)
        emoji = "🐝"
        text_id = self.canvas.create_text(x, y, text=emoji, font=("Arial", 14))
        self.bee_objects.append((text_id, bee))

    def draw_hive(self):
        hive_x, hive_y = self.hive_center
        self.canvas.create_text(hive_x, hive_y, text="🍯", font=("Arial", 40))

    def simulate_day(self):
        self.hive.simulate_day()

        self.label_day.config(text=f"День: {self.hive.day}")
        self.label_queens.config(text=f"Пчел: {len(self.hive.queens)}")
        self.label_drones.config(text=f"Трутней: {len(self.hive.drones)}")
        self.label_workers.config(text=f"Рабочих пчел: {len(self.hive.workers)}")
        self.label_larvae.config(text=f"Личинок: {len(self.hive.larvae)}")
        self.label_dead.config(text=f"Мертвых пчел: {len(self.hive.dead_bees)}")
        self.label_honey.config(text=f"Запасы меда: {int(self.hive.honey)}")


        if len(self.hive.workers) > 1:
            dead_bee = self.hive.workers.pop(0)
            dead_bee.die()
            self.hive.dead_bees.append(dead_bee)

            for _ in range(2):
                new_bee = Worker(self.hive.next_id, 5, 0)
                self.hive.workers.append(new_bee)
                self.hive.next_id += 1
                self.create_bee_visual(new_bee)

        current_ids = {b.id for _, b in self.bee_objects}
        for bee in self.hive.workers:
            if bee.id not in current_ids:
                self.create_bee_visual(bee)

    def animate(self):
        for text_id, bee in self.bee_objects[:]:
            if bee.alive:
                dx, dy = randint(-5, 5), randint(-5, 5)
                self.canvas.move(text_id, dx, dy)

                x, y = self.canvas.coords(text_id)
                if x < 0 or x > self.screen_width or y < 0 or y > self.screen_height:
                    self.canvas.move(text_id, -dx, -dy)
            else:
                self.canvas.delete(text_id)
                self.bee_objects.remove((text_id, bee))

        self.window.after(60, self.animate)

    def exit_program(self):
        self.window.destroy()

    def run(self):
        self.window.mainloop()


if __name__ == "__main__":
    app = VisualInterface()
    app.run()
