import requests
from bs4 import BeautifulSoup, SoupStrainer
import mysql.connector
from prettytable import PrettyTable
from openpyxl import Workbook
from openpyxl.utils.dataframe import dataframe_to_rows

def create_table(cursor):
    cursor.execute("""
    CREATE TABLE IF NOT EXISTS products (
        id INT AUTO_INCREMENT PRIMARY KEY,
        name VARCHAR(255) NOT NULL,
        amazon_url VARCHAR(1000) NOT NULL,
        flipkart_url VARCHAR(1000) NOT NULL,
        amazon_price DECIMAL(10, 2),
        flipkart_price DECIMAL(10, 2),
        amazon_rating DECIMAL(3, 2),
        amazon_reviews INT,
        flipkart_rating DECIMAL(3, 2),
        flipkart_reviews INT,
        comparison_result VARCHAR(50),
        last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
    )
    """)

def create_database(host, user, password, database_name):
    try:
        connection = mysql.connector.connect(
            host=host,
            user=user,
            password=password,
            autocommit=True
        )
        cursor = connection.cursor()
        
        try:
            cursor.execute(f"CREATE DATABASE {database_name}")
            print(f"Database '{database_name}' created successfully.")
        except mysql.connector.Error as e:
            if e.errno == 1007:
                print(f"Database '{database_name}' already exists. Continuing...")
            else:
                raise e

    except mysql.connector.Error as err:
        print(f"Error: {err}")

    finally:
        if 'connection' in locals():
            cursor.close()
            connection.close()


def insert_product(cursor, name, amazon_url, flipkart_url):
    cursor.execute("INSERT INTO products (name, amazon_url, flipkart_url) VALUES (%s, %s, %s)", (name, amazon_url, flipkart_url))

def view_products_with_prices(cursor):
    cursor.execute("SELECT id, name, amazon_price, flipkart_price, comparison_result, last_updated FROM products")
    products = cursor.fetchall()

    table = PrettyTable()
    table.field_names = ["ID", "Product Name", "Amazon Price (INR)", "Flipkart Price (INR)", "Price Comparison", "Last Updated"]

    for product in products:
        product_id, name, amazon_price, flipkart_price, price_comparison, last_updated = product
        table.add_row([product_id, name, amazon_price, flipkart_price, price_comparison, last_updated])

    print("Products with Prices:")
    print(table)

def view_products_with_urls(cursor):
    cursor.execute("SELECT id, name, amazon_url, flipkart_url, last_updated FROM products")
    products = cursor.fetchall()

    table = PrettyTable()
    table.field_names = ["ID", "Product Name", "Amazon URL", "Flipkart URL", "Last Updated"]

    for product in products:
        product_id, name, amazon_url, flipkart_url, last_updated = product
        table.add_row([product_id, name, amazon_url, flipkart_url, last_updated])

    print("Products with URLs:")
    print(table)

def view_products_with_ratings_and_reviews(cursor):
    cursor.execute("SELECT id, name, amazon_rating, amazon_reviews, flipkart_rating, flipkart_reviews FROM products")
    products = cursor.fetchall()

    table = PrettyTable()
    table.field_names = ["ID", "Product Name", "Amazon Rating", "Amazon Reviews", "Flipkart Rating", "Flipkart Reviews"]

    for product in products:
        product_id, name, amazon_rating, amazon_reviews, flipkart_rating, flipkart_reviews = product
        table.add_row([product_id, name, amazon_rating, amazon_reviews, flipkart_rating, flipkart_reviews])

    print("Products with Ratings and Reviews:")
    print(table)

def update_prices(cursor):
    cursor.execute("SELECT id, name, amazon_url, flipkart_url FROM products")
    products = cursor.fetchall()

    for product in products:
        product_id, name, amazon_url, flipkart_url = product
        try:
            # Update Amazon price
            amazon_price_filter = SoupStrainer("span", class_="a-price-whole")
            amazon_rating_filter = SoupStrainer("span", class_="a-icon-alt")
            amazon_reviews_filter = SoupStrainer("span", id="acrCustomerReviewText")

            response = requests.get(amazon_url, headers=({"User-Agent": "Your-User-Agent", 'Accept-Language': 'en-US, en;q=0.5'}))
            soup = BeautifulSoup(response.content, "html.parser", parse_only=amazon_price_filter)
            price_element = soup.find("span", class_="a-price-whole")

            soup = BeautifulSoup(response.content, "html.parser", parse_only=amazon_rating_filter)
            amazon_rating_element = soup.find("span", class_="a-icon-alt")

            soup = BeautifulSoup(response.content, "html.parser", parse_only=amazon_reviews_filter)
            amazon_reviews_element = soup.find("span", id="acrCustomerReviewText")

            if price_element:
                amazon_price = float(price_element.get_text().strip().replace("₹", "").replace(",", ""))
            else:
                amazon_price = None

            if amazon_rating_element:
                amazon_rating = amazon_rating_element.get_text().strip().split(' out of')[0]
            else:
                amazon_rating = None

            if amazon_reviews_element:
                amazon_reviews = int(amazon_reviews_element.get_text().strip().replace(",", "").split(' ')[0])
            else:
                amazon_reviews = None

            # Update Flipkart price
            flipkart_price_filter = SoupStrainer("div", class_="_30jeq3 _16Jk6d")
            flipkart_rating_filter = SoupStrainer("div", class_="_3LWZlK")
            flipkart_reviews_filter = SoupStrainer("span", class_="_2_R_DZ")

            response = requests.get(flipkart_url, headers=({"User-Agent": "Your-User-Agent", 'Accept-Language': 'en-US, en;q=0.5'}))
            soup = BeautifulSoup(response.content, "html.parser", parse_only=flipkart_price_filter)
            price_element = soup.find("div", class_="_30jeq3 _16Jk6d")

            soup = BeautifulSoup(response.content, "html.parser", parse_only=flipkart_rating_filter)
            flipkart_rating_element = soup.find("div", class_="_3LWZlK")

            soup = BeautifulSoup(response.content, "html.parser", parse_only=flipkart_reviews_filter)
            flipkart_reviews_element = soup.find("span", class_="_2_R_DZ")

            if price_element:
                flipkart_price = float(price_element.get_text().strip().replace("₹", "").replace(",", ""))
            else:
                flipkart_price = None

            if flipkart_rating_element:
                flipkart_rating = flipkart_rating_element.get_text().strip()
            else:
                flipkart_rating = None

            if flipkart_reviews_element:
                flipkart_reviews_text = ''.join(filter(str.isdigit, flipkart_reviews_element.get_text().strip()))
                flipkart_reviews = int(flipkart_reviews_text)
            else:
                flipkart_reviews = None

            comparison_result = "Amazon is cheaper" if amazon_price < flipkart_price else "Flipkart is cheaper" if flipkart_price < amazon_price else "Prices are the same"

            cursor.execute("UPDATE products SET amazon_price = %s, flipkart_price = %s, amazon_rating = %s, amazon_reviews = %s, flipkart_rating = %s, flipkart_reviews = %s, comparison_result = %s WHERE id = %s",
                           (amazon_price, flipkart_price, amazon_rating, amazon_reviews, flipkart_rating, flipkart_reviews, comparison_result, product_id))
            
            print(f"Updating Amazon price for {name}...")
            if amazon_price is not None:
                print(f"Amazon Price for {name}: ₹{amazon_price}")
                print(f"Amazon Rating for {name}: {amazon_rating}")
                print(f"Amazon Reviews for {name}: {amazon_reviews}")

            print(f"Updating Flipkart price for {name}...")
            if flipkart_price is not None:
                print(f"Flipkart Price for {name}: ₹{flipkart_price}")
                print(f"Flipkart Rating for {name}: {flipkart_rating}")
                print(f"Flipkart Reviews for {name}: {flipkart_reviews}")

        except Exception as e:
            print(f"Error updating prices for {name}: {e}")

        print(f"Updating prices for {name} completed.")

def delete_product(cursor, product_id):
    cursor.execute("DELETE FROM products WHERE id = %s", (product_id,))
    print(f"Product with ID {product_id} deleted successfully.")

def search_product(cursor, search_term):
    cursor.execute("SELECT id, name, amazon_price, flipkart_price, amazon_rating, flipkart_rating, amazon_reviews, flipkart_reviews, last_updated FROM products WHERE name LIKE %s", (f"%{search_term}%",))
    products = cursor.fetchall()

    table = PrettyTable()
    table.field_names = ["ID", "Product Name", "Amazon Price (INR)", "Flipkart Price (INR)", "Amazon Rating", "Flipkart Rating", "Amazon Reviews", "Flipkart Reviews", "Last Updated"]

    for product in products:
        product_id, name, amazon_price, flipkart_price, amazon_rating, flipkart_rating, amazon_reviews, flipkart_reviews, last_updated = product
        table.add_row([product_id, name, amazon_price, flipkart_price, amazon_rating, flipkart_rating, amazon_reviews, flipkart_reviews, last_updated])

    print(f"Search Results for '{search_term}':")
    print(table)


def export_to_excel(cursor, excel_file_name, selected_columns):
    cursor.execute("SELECT ID, name, amazon_price, flipkart_price, last_updated FROM products")
    products = cursor.fetchall()

    workbook = Workbook()
    sheet = workbook.active

    # Add headers
    sheet.append(selected_columns)

    for product in products:
        sheet.append(product)

    workbook.save(excel_file_name)
    print(f"Data exported to {excel_file_name}")

def main():
    db_host = input("Enter the database host: ")
    db_user = input("Enter the database user: ")
    db_password = input("Enter the database password: ")
    db_name = "products"

    create_database(db_host, db_user, db_password, db_name)

    db = mysql.connector.connect(
        host=db_host,
        user=db_user,
        password=db_password,
        database=db_name,
        autocommit=True
    )
    cursor = db.cursor()

    create_table(cursor)

    while True:
        print("1. Add Product")
        print("2. Update Prices")
        print("3. View Products with Prices")
        print("4. View Products with URLs only")
        print("5. View Products with ratings and reviews of Flipkart and Amazon together")
        print("6. Delete Product")
        print("7. Search for a Product")
        print("8. Export Data to Excel")
        print("9. Exit")
        choice = input("Enter your choice: ")

        if choice == "1":
            name = input("Enter product name: ")
            amazon_url = input("Enter Amazon product URL: ")
            flipkart_url = input("Enter Flipkart product URL: ")
            insert_product(cursor, name, amazon_url, flipkart_url)
        elif choice == "2":
            update_prices(cursor)
        elif choice == "3":
            view_products_with_prices(cursor)
        elif choice == "4":
            view_products_with_urls(cursor)
        elif choice == "5":
            view_products_with_ratings_and_reviews(cursor)
        elif choice == "6":
            product_id = input("Enter the ID of the product to delete: ")
            delete_product(cursor, product_id)
        elif choice == "7":
            search_term = input("Enter a product name to search for: ")
            search_product(cursor, search_term)
        elif choice == "8":
            excel_file_name = input("Enter the Excel file name (e.g., data.xlsx): ")
            if not excel_file_name.endswith(".xlsx"):
                excel_file_name += ".xlsx"
            selected_columns = ["ID", "Product Name", "Amazon Price (INR)", "Flipkart Price (INR)", "Last Updated"]
            export_to_excel(cursor, excel_file_name, selected_columns)
        elif choice == "9":
            break

    db.close()

if __name__ == "__main__":
    main()
