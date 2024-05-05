# PythonMiniProject
import tkinter as tk
from tkinter import ttk, messagebox, scrolledtext
import requests

class IceCreamParlor:
    def __init__(self, master):
        self.master = master
        self.master.title("pcube Ice Cream Parlor")
        self.master.geometry("800x500")
        self.master.resizable(False, False)
        self.flavors = {"Chocolate": 150.0, "Vanilla": 120.0, "Strawberry": 135.0}
        self.toppings = {"Sprinkles": 37.5, "Choc Chips": 56.25, "Caramel": 75.0}
        self.sizes = {"Small": 75.0, "Medium": 112.5, "Large": 150.0}
        self.selected_flavor = tk.StringVar(value="Chocolate")
        self.selected_toppings = set()
        self.selected_size = tk.StringVar(value="Small")

        self.scoops_var = tk.IntVar(value=1)
        self.create_widgets()
    def create_widgets(self):
        # Left
        left_frame = ttk.Frame(self.master)
        left_frame.grid(row=0, column=0, padx=10, pady=10, sticky="n")

        # Flavor selection
        flavor_frame = ttk.LabelFrame(left_frame, text="Select Flavor", padding=(10, 5))
        flavor_frame.pack(pady=10, padx=10, fill="both")

        for flavor, cost in self.flavors.items():
            ttk.Radiobutton(flavor_frame, text=f"{flavor} (₹{cost:.2f})", variable=self.selected_flavor, value=flavor,
                            command=self.update_receipt).pack(anchor="w", padx=5)
        # Toppings selection
        toppings_frame = ttk.LabelFrame(left_frame, text="Select Toppings", padding=(10, 5))
        toppings_frame.pack(pady=10, padx=10, fill="both")
        for topping, cost in self.toppings.items():
            ttk.Checkbutton(toppings_frame, text=f"{topping} (+₹{cost:.2f})", variable=tk.BooleanVar(),
             command=lambda t=topping: self.toggle_topping(t)).pack(anchor="w", padx=5)

        # Size selection
        size_frame = ttk.LabelFrame(left_frame, text="Select Size", padding=(10, 5))
        size_frame.pack(pady=10, padx=10, fill="both")
        for size, cost in self.sizes.items():
            ttk.Radiobutton(size_frame, text=f"{size} (+₹{cost:.2f})", variable=self.selected_size, value=size,
                            command=self.update_receipt).pack(anchor="w", padx=5)

        # Scoops selection
        scoops_frame = ttk.LabelFrame(left_frame, text="Number of Scoops", padding=(10, 5))
        scoops_frame.pack(pady=10, padx=10, fill="both")
        ttk.Spinbox(scoops_frame, from_=1, to=5, textvariable=self.scoops_var, command=self.update_receipt).pack(anchor="w", padx=5)
        # Right
        right_frame = ttk.Frame(self.master)
        right_frame.grid(row=0, column=1, padx=10, pady=10, sticky="n")

        receipt_frame = ttk.LabelFrame(right_frame, text="Receipt", padding=(10, 5))
        receipt_frame.pack(pady=10, padx=10, fill="both", expand=True)

        self.receipt_text = scrolledtext.ScrolledText(receipt_frame, height=10, width=40, wrap=tk.WORD, state=tk.DISABLED)
        self.receipt_text.pack(pady=5, fill=tk.BOTH, expand=True)

        # Total Cost
        total_frame = ttk.Frame(right_frame, padding=(10, 5))
        total_frame.pack(pady=10, padx=10, fill="both")

        ttk.Label(total_frame, text="Total Cost:", font=("Arial", 12)).pack(side="left", padx=5)
        self.total_label = ttk.Label(total_frame, text="₹0.00", font=("Arial", 12, "bold"), foreground="green")
        self.total_label.pack(side="left", padx=5)

        # Place Order button
        ttk.Button(right_frame, text="Place Order", command=self.place_order).pack(pady=20)

    def toggle_topping(self, topping):
        if topping in self.selected_toppings:
            self.selected_toppings.remove(topping)
        else:
            self.selected_toppings.add(topping)
        self.update_receipt()
    def update_receipt(self):
        receipt_content = f"Flavor: {self.selected_flavor.get()}\n"
        receipt_content += f"Toppings: {', '.join(self.selected_toppings)}\n"
        receipt_content += f"Size: {self.selected_size.get()}\n"
        receipt_content += f"Scoops: {self.scoops_var.get()}\n"
        flavor_cost = self.flavors[self.selected_flavor.get()]
        toppings_cost = sum(self.toppings[topping] for topping in self.selected_toppings)
        size_cost = self.sizes[self.selected_size.get()]
        scoops_cost = 37.5 * (self.scoops_var.get() - 1)
        total_cost = flavor_cost + toppings_cost + size_cost + scoops_cost
        receipt_content += f"\nTotal Cost: ₹{total_cost:.2f}"
        self.receipt_text.config(state=tk.NORMAL)
        self.receipt_text.delete(1.0, tk.END)
        self.receipt_text.insert(tk.END, receipt_content)
        self.receipt_text.config(state=tk.DISABLED)
        self.total_label.config(text=f"₹{total_cost:.2f}")
    def place_order(self):
        flavor = self.selected_flavor.get()
        size = self.selected_size.get()
        scoops = self.scoops_var.get()
        scoops_cost = 37.5 * (scoops - 1) if scoops > 1 else 0.0

        data = {
            'flavor': flavor,
            'toppings': list(self.selected_toppings),
            'size': size,
            'scoops': scoops,
            'scoops_cost': scoops_cost
        }

        try:
            response = requests.post('http://localhost:5000/place_order', json=data)

            if response.status_code == 200:
                result_message = response.json()['message']
                messagebox.showinfo("Order Placed", result_message)
                self.reset_selections()
            else:
                messagebox.showerror("Error", "Failed to place order. Please try again.")
        except requests.ConnectionError:
            messagebox.showerror("Error", "Failed to connect to the server. Make sure your backend is running.")

    def reset_selections(self):
        self.selected_flavor.set("Chocolate")
        self.selected_toppings.clear()
        self.selected_size.set("Small")
        self.scoops_var.set(1)

        self.receipt_text.config(state=tk.NORMAL)
        self.receipt_text.delete(1.0, tk.END)
        self.receipt_text.config(state=tk.DISABLED)

        self.total_label.config(text="₹0.00")

if __name__ == "__main__":
    root = tk.Tk()
    app = IceCreamParlor(root)
    root.mainloop()
