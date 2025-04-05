[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/_-JrKZbN)
CREATE TABLE Product (
    ProductID INT PRIMARY KEY,
    ProductName VARCHAR(100),
    Price DECIMAL(10, 2),
    Quantity INT
);
import java.sql.*;
import java.util.Scanner;

public class ProductCRUDApp {
    private static final String DB_URL = "jdbc:mysql://localhost:3306/your_database";
    private static final String USER = "your_username";
    private static final String PASS = "your_password";

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        try (Connection conn = DriverManager.getConnection(DB_URL, USER, PASS)) {
            conn.setAutoCommit(false); // Transaction control

            while (true) {
                System.out.println("\n=== Product Management ===");
                System.out.println("1. Add Product");
                System.out.println("2. View Products");
                System.out.println("3. Update Product");
                System.out.println("4. Delete Product");
                System.out.println("5. Exit");
                System.out.print("Choose an option: ");
                int choice = scanner.nextInt();

                switch (choice) {
                    case 1:
                        addProduct(conn, scanner);
                        break;
                    case 2:
                        viewProducts(conn);
                        break;
                    case 3:
                        updateProduct(conn, scanner);
                        break;
                    case 4:
                        deleteProduct(conn, scanner);
                        break;
                    case 5:
                        System.out.println("Exiting...");
                        conn.close();
                        scanner.close();
                        return;
                    default:
                        System.out.println("Invalid option. Try again.");
                }
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    private static void addProduct(Connection conn, Scanner scanner) {
        try {
            System.out.print("Enter Product ID: ");
            int id = scanner.nextInt();
            scanner.nextLine(); // consume newline
            System.out.print("Enter Product Name: ");
            String name = scanner.nextLine();
            System.out.print("Enter Price: ");
            double price = scanner.nextDouble();
            System.out.print("Enter Quantity: ");
            int qty = scanner.nextInt();

            String sql = "INSERT INTO Product (ProductID, ProductName, Price, Quantity) VALUES (?, ?, ?, ?)";
            try (PreparedStatement stmt = conn.prepareStatement(sql)) {
                stmt.setInt(1, id);
                stmt.setString(2, name);
                stmt.setDouble(3, price);
                stmt.setInt(4, qty);

                int rows = stmt.executeUpdate();
                conn.commit();
                System.out.println(rows + " product(s) added.");
            }
        } catch (SQLException e) {
            try {
                conn.rollback();
                System.out.println("Transaction rolled back.");
            } catch (SQLException ex) {
                ex.printStackTrace();
            }
            e.printStackTrace();
        }
    }

    private static void viewProducts(Connection conn) {
        String sql = "SELECT * FROM Product";
        try (Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery(sql)) {

            System.out.println("\nProduct List:");
            while (rs.next()) {
                System.out.printf("ID: %d, Name: %s, Price: %.2f, Qty: %d%n",
                        rs.getInt("ProductID"),
                        rs.getString("ProductName"),
                        rs.getDouble("Price"),
                        rs.getInt("Quantity"));
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    private static void updateProduct(Connection conn, Scanner scanner) {
        try {
            System.out.print("Enter Product ID to update: ");
            int id = scanner.nextInt();
            scanner.nextLine(); // consume newline
            System.out.print("Enter new Product Name: ");
            String name = scanner.nextLine();
            System.out.print("Enter new Price: ");
            double price = scanner.nextDouble();
            System.out.print("Enter new Quantity: ");
            int qty = scanner.nextInt();

            String sql = "UPDATE Product SET ProductName=?, Price=?, Quantity=? WHERE ProductID=?";
            try (PreparedStatement stmt = conn.prepareStatement(sql)) {
                stmt.setString(1, name);
                stmt.setDouble(2, price);
                stmt.setInt(3, qty);
                stmt.setInt(4, id);

                int rows = stmt.executeUpdate();
                conn.commit();
                System.out.println(rows + " product(s) updated.");
            }
        } catch (SQLException e) {
            try {
                conn.rollback();
                System.out.println("Transaction rolled back.");
            } catch (SQLException ex) {
                ex.printStackTrace();
            }
            e.printStackTrace();
        }
    }

    private static void deleteProduct(Connection conn, Scanner scanner) {
        try {
            System.out.print("Enter Product ID to delete: ");
            int id = scanner.nextInt();

            String sql = "DELETE FROM Product WHERE ProductID=?";
            try (PreparedStatement stmt = conn.prepareStatement(sql)) {
                stmt.setInt(1, id);

                int rows = stmt.executeUpdate();
                conn.commit();
                System.out.println(rows + " product(s) deleted.");
            }
        } catch (SQLException e) {
            try {
                conn.rollback();
                System.out.println("Transaction rolled back.");
            } catch (SQLException ex) {
                ex.printStackTrace();
            }
            e.printStackTrace();
        }
    }
}
