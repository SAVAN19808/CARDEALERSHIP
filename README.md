import mysql.connector
from tabulate import tabulate

mycon = mysql.connector.connect(
    host="localhost",
    user="root",
    password="root",
    database="cardealership"
)

table_map = {
    'a': 'cars',
    'b': 'customer',
    'c': 'employee',
    'd': 'sale_history',
    'e': 'purchase_history'
}

if mycon.is_connected():
    print("Connection Successful")
    cursor = mycon.cursor()

    while True:
        print("\nMAIN MENU")
        print("1. Insert Data")
        print("2. View Data")
        print("3. Update Data")
        print("4. Delete Data")
        print("5. Exit")
        menu_choice = input("Enter your choice (1-5): ")

        if menu_choice == '1':
            print("\nInsert into:")
            print("a. Cars")
            print("b. Customer")
            print("c. Employee")
            print("d. Sale History")
            print("e. Purchase History")
            key = input("Enter choice (a-e): ").lower()

            if key in table_map:
                tname = table_map[key]
                try:
                    cursor.execute(f"SELECT * FROM {tname}")
                    data = cursor.fetchall()
                    cursor.execute(f"DESC {tname}")
                    headers = [col[0] for col in cursor.fetchall()]
                    print(f"\nCurrent records in '{tname}':")
                    print(tabulate(data, headers=headers, tablefmt="simple"))
                except Exception as e:
                    print(f"Error fetching data: {e}")

            if key == 'a':
                q = "INSERT INTO cars VALUES (%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s)"
                val = (
                    input("Car Number: "), input("Brand Name: "), input("Model Name: "),
                    input("Fuel Type: "), input("Date of Manufacture (YYYY-MM-DD): "),
                    input("Transmission: "), input("Body Type: "),
                    int(input("Kilometers Driven: ")), int(input("Price: ")),
                    input("Color: "), input("Condition: ")
                )

            elif key == 'b':
                q = "INSERT INTO customer VALUES (%s,%s,%s,%s,%s,%s,%s)"
                val = (
                    input("Customer ID: "), input("Name: "), input("Address: "),
                    input("Phone: "), input("Payment Mode: "),
                    input("Email: "), input("Date of Birth (YYYY-MM-DD): ")
                )

            elif key == 'c':
                q = "INSERT INTO employee VALUES (%s,%s,%s,%s,%s,%s,%s)"
                val = (
                    input("Employee ID: "), input("Name: "),
                    input("Date of Joining (YYYY-MM-DD): "), input("Salary: "),
                    input("Job Title: "), input("Address: "), input("Email: ")
                )

            elif key == 'd':
                q = "INSERT INTO sale_history VALUES (%s,%s,%s,%s,%s,%s,%s,%s,%s)"
                val = (
                    input("Buyer Name: "), input("Buyer Address: "), input("Phone: "),
                    input("Email: "), input("Mode of Transaction: "),
                    input("Sold Car Number: "), input("Car Model: "),
                    input("Date Car Sold (YYYY-MM-DD): "), input("Price of Car: ")
                )

            elif key == 'e':
                q = "INSERT INTO purchase_history VALUES (%s,%s,%s,%s,%s,%s,%s,%s,%s)"
                val = (
                    input("Seller Name: "), input("Seller Address: "), input("Phone: "),
                    input("Email: "), input("Bought Car Number: "),
                    input("Car Model: "), input("Date Car Bought (YYYY-MM-DD): "),
                    input("Car Range (km): "), input("Price: ")
                )
            else:
                print("Invalid table option.")
                continue

            try:
                cursor.execute(q, val)
                mycon.commit()
                print("Data inserted successfully.")
                # Show updated table after insert
                cursor.execute(f"SELECT * FROM {table_map[key]}")
                data = cursor.fetchall()
                cursor.execute(f"DESC {table_map[key]}")
                headers = [col[0] for col in cursor.fetchall()]
                print(f"\nUpdated records in '{table_map[key]}':")
                print(tabulate(data, headers=headers, tablefmt="simple"))
            except Exception as e:
                print(f"Error inserting data: {e}")

        elif menu_choice == '2':
            print("\nView Table:")
            print("a. Cars")
            print("b. Customer")
            print("c. Employee")
            print("d. Sale History")
            print("e. Purchase History")
            key = input("Enter choice (a-e): ").lower()

            if key in table_map:
                tname = table_map[key]
                try:
                    cursor.execute(f"SELECT * FROM {tname}")
                    data = cursor.fetchall()
                    cursor.execute(f"DESC {tname}")
                    headers = [col[0] for col in cursor.fetchall()]
                    print(f"\nRecords in '{tname}':")
                    print(tabulate(data, headers=headers, tablefmt="simple"))
                except Exception as e:
                    print(f"Error fetching data: {e}")
            else:
                print("Invalid choice.")

        elif menu_choice == '3':
            print("\nUpdate Table:")
            print("a. Cars")
            print("b. Customer")
            print("c. Employee")
            print("d. Sale History")
            print("e. Purchase History")
            key = input("Enter choice (a-e): ").lower()

            if key in table_map:
                # Show current data before update
                try:
                    cursor.execute(f"SELECT * FROM {table_map[key]}")
                    cursor.execute(f"DESC {table_map[key]}")
                    headers = [col[0] for col in cursor.fetchall()]
                    print(f"\nCurrent records in '{table_map[key]}':")
                except Exception as e:
                    print(f"Error fetching data: {e}")

                cursor.execute(f"DESC {table_map[key]}")
                columns = [col[0] for col in cursor.fetchall()]

                print(f"\nColumns in {table_map[key]}:")
                for i, col in enumerate(columns):
                    print(f"{i + 1}. {col}")

                try:
                    cond_val = input(f"Enter value to match for condition column: ")

                    count = int(input("How many columns do you want to update? "))
                    updates = []
                    values = []

                    for i in range(count):
                        upd_index = int(input(f"Enter column number to update ({i + 1}/{count}): ")) - 1
                        new_val = input(f"Enter new value for '{columns[upd_index]}': ")
                        updates.append(f"{columns[upd_index]} = %s")
                        values.append(new_val)

                    # Instead, ask for condition column name directly:
                    cond_column_name = input("Enter condition column name: ")
                    q = f"UPDATE {table_map[key]} SET {', '.join(updates)} WHERE LOWER({cond_column_name}) = LOWER(%s)"
                    values.append(cond_val)
                    cursor.execute(q, tuple(values))
                    mycon.commit()

                    if cursor.rowcount == 0:
                        print("No matching record found. Nothing was updated.")
                    else:
                        print(f"{cursor.rowcount} record(s) updated.")
                        # Show updated table after update
                        cursor.execute(f"SELECT * FROM {table_map[key]}")
                        cursor.execute(f"DESC {table_map[key]}")
                        headers = [col[0] for col in cursor.fetchall()]
                        print(f"\nUpdated records in '{table_map[key]}':")
                except Exception as e:
                    print(f"Error updating data: {e}")
            else:
                print("Invalid choice.")

        elif menu_choice == '4':
            print("\nDelete from Table:")
            print("a. Cars")
            print("b. Customer")
            print("c. Employee")
            print("d. Sale History")
            print("e. Purchase History")
            print("f. Fire an Employee")  # New option
            key = input("Enter choice (a-e or f): ").lower()

            if key == 'f':
                emp_id = input("Enter Employee ID to fire: ")
                try:
                    cursor.execute("DELETE FROM employee WHERE LOWER(employeeID) = LOWER(%s)", (emp_id,))
                    mycon.commit()
                    if cursor.rowcount == 0:
                        print("No matching employee found. No one was fired.")
                    else:
                        print(f"Employee with ID '{emp_id}' has been fired and removed from records.")
                        # Show updated employee table
                        cursor.execute("SELECT * FROM employee")
                        data = cursor.fetchall()
                        cursor.execute("DESC employee")
                        headers = [col[0] for col in cursor.fetchall()]
                        print("\nUpdated Employee Records:")
                        print(tabulate(data, headers=headers, tablefmt="simple"))
                except Exception as e:
                    print(f"Error firing employee: {e}")

            elif key in table_map:
                tname = table_map[key]
                # Show current data before delete
                try:
                    cursor.execute(f"SELECT * FROM {tname}")
                    data = cursor.fetchall()
                    cursor.execute(f"DESC {tname}")
                    headers = [col[0] for col in cursor.fetchall()]
                    print(f"\nCurrent records in '{tname}':")
                    print(tabulate(data, headers=headers, tablefmt="simple"))
                except Exception as e:
                    print(f"Error fetching data: {e}")

                column = input("Enter column name to match for deletion: ")
                value = input(f"Enter value to match for '{column}': ")
                try:
                    q = f"DELETE FROM {tname} WHERE LOWER({column}) = LOWER(%s)"
                    cursor.execute(q, (value,))
                    mycon.commit()
                    if cursor.rowcount == 0:
                        print("No matching record found. Nothing was deleted.")
                    else:
                        print("Record deleted.")
                        # Show updated table after delete
                        cursor.execute(f"SELECT * FROM {tname}")
                        data = cursor.fetchall()
                        cursor.execute(f"DESC {tname}")
                        headers = [col[0] for col in cursor.fetchall()]
                        print(f"\nUpdated records in '{tname}':")
                        print(tabulate(data, headers=headers, tablefmt="simple"))
                except Exception as e:
                    print(f"Error deleting record: {e}")
            else:
                print("Invalid choice.")

        elif menu_choice == '5':
            break

        cont = input("\nDo you want to continue? (yes/no): ").lower()
        if cont != 'yes':
            last_action = input("Did you BUY or SELL a car? (buy/sell/none): ").lower()

            if last_action == 'buy':
                carnum = input("Enter Bought Car Number: ")
                try:
                    cursor.execute("SELECT * FROM purchase_history WHERE LOWER(boughtcarNumber) = LOWER(%s)", (carnum,))
                    result = cursor.fetchall()
                    if result:
                        cursor.execute("DESC purchase_history")
                        headers = [col[0] for col in cursor.fetchall()]
                        print("\n🧾 PURCHASE BILL")
                        print(tabulate(result, headers=headers, tablefmt="grid"))
                    else:
                        print("No matching purchase found.")
                except Exception as e:
                    print(f"Error fetching purchase bill: {e}")

            elif last_action == 'sell':
                carnum = input("Enter Sold Car Number: ")
                try:
                    cursor.execute("SELECT * FROM sale_history WHERE LOWER(soldcarNumber) = LOWER(%s)", (carnum,))
                    result = cursor.fetchall()
                    if result:
                        cursor.execute("DESC sale_history")
                        headers = [col[0] for col in cursor.fetchall()]
                        print("\n🧾 SALE BILL")
                        print(tabulate(result, headers=headers, tablefmt="grid"))
                    else:
                        print("No matching sale found.")
                except Exception as e:
                    print(f"Error fetching sale bill: {e}")

            print("Thank you! Exiting now.")
            break
        else:
            print("Continuing to main menu...")

    cursor.close()
    mycon.close()
    print("Connection closed.")
