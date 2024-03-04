# python-project-using-tkinter
import sqlite3
from tkinter import *
from tkinter import messagebox
import webbrowser

# Function to create SQLite database and table
def create_database():
    conn = sqlite3.connect("stock_prices.db")
    c = conn.cursor()
    c.execute('''CREATE TABLE IF NOT EXISTS stocks
                 (id INTEGER PRIMARY KEY,
                 name TEXT NOT NULL,
                 price INTEGER NOT NULL)''')
    conn.commit()
    conn.close()

# Function to populate SQLite table with initial stock prices
def populate_database():
    conn = sqlite3.connect("stock_prices.db")
    c = conn.cursor()
    initial_stock_prices = {
        "Reliance": 100,
        "Mahendra": 150,
        "Adani": 80,
        "Tata": 120,
        "D-Mart": 90,
        "Midkind": 110,
        "Infosys": 70
    }
    for name, price in initial_stock_prices.items():
        c.execute("INSERT INTO stocks (name, price) VALUES (?, ?)", (name, price))
    conn.commit()
    conn.close()

# Function to retrieve stock prices from SQLite database
def retrieve_stock_prices():
    conn = sqlite3.connect("stock_prices.db")
    c = conn.cursor()
    c.execute("SELECT name, price FROM stocks")
    stocks = c.fetchall()
    conn.close()
    return dict(stocks)

# Function to open new page for stock transactions
def open_new_page(stock_name):
    new_window = Toplevel(root)
    new_window.title("US STOCK TRAINER APP")  # Set title
    new_window.attributes('-fullscreen', True)  # Set fullscreen

    def calculate_total(transaction_type):
        # Function to calculate total amount and show message box
        try:
            price = stock_prices.get(stock_name, 0)
            quantity = int(quantity_entry.get())
            if quantity <= 0:
                raise ValueError("Quantity should be greater than zero.")
            total = price * quantity
            message = f"{transaction_type} - {stock_name} - Quantity: {quantity} - Price: ${price} - Total Price: ${total}"
            messagebox.showinfo("Transaction", message)
            update_display(message)
        except ValueError as e:
            messagebox.showerror("Error", str(e))

    def calculate_total_price():
        # Function to calculate total price (stock price * stock quantity)
        try:
            price = stock_prices.get(stock_name, 0)
            quantity = int(quantity_entry.get())
            if quantity <= 0:
                raise ValueError("Quantity should be greater than zero.")
            total_price = price * quantity
            update_display("Total Price: ${:.2f}".format(total_price))
        except ValueError as e:
            messagebox.showerror("Error", str(e))

    def update_display(message):
        # Function to update display with transaction details
        display_label.config(text=message)

    # Create widgets for the new page
    price_label = Label(new_window, text=f"Stock Price: ${stock_prices.get(stock_name, 0)}", font=("Arial", 16))
    price_label.grid(row=0, column=0, columnspan=2, pady=20)

    quantity_label = Label(new_window, text="Enter quantity:", font=("Arial", 16))
    quantity_label.grid(row=1, column=0, pady=20)

    quantity_entry = Entry(new_window, font=("Arial", 16))
    quantity_entry.grid(row=1, column=1, pady=20)

    total_price_button = Button(new_window, text="Total Price", bg="blue", fg="white", font=("Arial", 16), command=calculate_total_price)
    total_price_button.grid(row=2, column=0, columnspan=2, padx=20, pady=20)

    buy_button = Button(new_window, text="Buy", bg="green", fg="white", font=("Arial", 16), command=lambda: calculate_total("Buy"))
    buy_button.grid(row=3, column=0, padx=20, pady=20)

    sell_button = Button(new_window, text="Sell", bg="red", fg="white", font=("Arial", 16), command=lambda: calculate_total("Sell"))
    sell_button.grid(row=3, column=1, padx=20, pady=20)

    display_label = Label(new_window, text="", font=("Arial", 18))
    display_label.grid(row=4, column=0, columnspan=2, pady=20)

    quit_button = Button(new_window, text="Quit", bg="red", fg="white", font=("Arial", 16), command=new_window.destroy)
    quit_button.grid(row=0, column=3, padx=20, pady=20, sticky="ne")

    # Label for currency conversion rate
    conversion_rate_label = Label(new_window, text="1$ = 82.85 ₹", font=("Arial", 12, "bold"), bg="white", fg="black")
    conversion_rate_label.grid(row=5, column=0, columnspan=4, pady=20, sticky="se")

# Function to open information page
def open_info_page(stock_name):
    google_search_url = f"https://www.google.com/search?q={stock_name}+stock+info"
    webbrowser.open_new(google_search_url)

# Create the main window
root = Tk()
root.title("US STOCK TRAINER APP")  # Set the title of the first page
root.geometry("800x600")  # Increased window size
root.configure(bg="white")

# Create SQLite database and table if not exists
create_database()
populate_database()
stock_prices = retrieve_stock_prices()

# Main window components
heading_label = Label(root, text="US STOCK TRAINER APP", font=("Arial", 20, "bold"), bg="white", fg="black")
heading_label.grid(row=0, column=0, padx=20, pady=10)

stock_name_label = Label(root, text="Stock Name:", font=("Arial", 16), bg="white", fg="black")
stock_name_label.grid(row=2, column=0, pady=(0, 5), padx=20)

options = ["Reliance", "Mahendra", "Adani", "Tata", "D-Mart", "Midkind", "Infosys"]

menu_button = Menubutton(root, text="Stock Info", bg="blue", fg="white", font=("Arial", 16))
menu_button.menu = Menu(menu_button, tearoff=0)
menu_button["menu"] = menu_button.menu

for i, option in enumerate(options):
    menu_button.menu.add_command(label=option, command=lambda opt=option: open_info_page(opt))
    button = Button(root, text=option, command=lambda opt=option: open_new_page(opt), bg="yellow", fg="black", font=("Arial", 16))
    button.grid(row=3+i, column=0, pady=10, padx=20, sticky="ew")

separator = Label(root, text="---------------------", bg="white", fg="black")
separator.grid(row=4+len(options), column=0, pady=5, padx=20, sticky="ew")

menu_button.grid(row=5+len(options), column=0,pady=10)

# Label for currency conversion rate
conversion_rate_label = Label(root, text="1$ = 82.85 ₹", font=("Arial", 12, "bold"), bg="white", fg="black")
conversion_rate_label.grid(row=6+len(options), column=0, columnspan=2, pady=10, sticky="se")

root.mainloop()

