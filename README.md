// Streamlined and optimized UnlaceApp.java
import java.io.*;
import java.util.*;
import java.time.*;
import java.time.format.*;
import java.util.function.*;
import java.util.stream.*;

public class UnlaceApp { // Main class for the Unlace shoe inventory system
    private static final String DATA_FILE = "store_data.ser"; // File name to store inventory and sales data
    private static final DateTimeFormatter DATE_FORMAT = DateTimeFormatter.ofPattern("MM/dd/yyyy"); // Formatter for short date
    private static final DateTimeFormatter DATE_TIME_FORMAT = DateTimeFormatter.ofPattern("MM/dd/yyyy HH:mm"); // Formatter for full date and time
    private static final int MAX_PRICE = 10000; // Maximum allowed price for a shoe
    private static final int MAX_QUANTITY = 1000; // Maximum allowed quantity for stock

    static class User implements Serializable { // Represents a user who logs in
        final String username; // User's name
        final String email; // User's email address
        final LocalDateTime loginTime; // Timestamp when the user logged in

        public User(String username, String email) { // Constructor to set name, email, and login time
            this.username = username; // Set the user's name
            this.email = email; // Set the user's email
            this.loginTime = LocalDateTime.now(); // Capture current time as login time
        }
    }

    static abstract class Record implements Serializable { // Abstract base class for shared record fields
        private static int nextId = 1; // Static counter to assign unique IDs
        final int id; // Unique ID for each record
        String name; // Name of the shoe or sale item
        int quantity; // Number of items in stock or sold
        final LocalDateTime dateAdded; // When the record was created

        public Record(String name, int quantity) { // Constructor for shared fields
            this.id = nextId++; // Assign and increment the record ID
            this.name = name; // Set name
            this.quantity = quantity; // Set quantity
            this.dateAdded = LocalDateTime.now(); // Record creation time
        }
    }

    public static class ShoeItem extends Record { // Subclass representing a shoe in the inventory
        double price; // Price of the shoe
        String description; // Optional description of the shoe
        String seller; // Seller of the shoe
        String category; // Category of the shoe (e.g., Running, Formal)
        String size; // Size of the shoe

        public ShoeItem(String name, double price, int quantity, String seller, String category, String size) { // Constructor for ShoeItem
            super(name, quantity); // Call to base Record constructor
            this.price = price;
            this.seller = seller;
            this.category = category;
            this.size = size;
            this.description = ""; // Initialize empty description
        }

        @Override
        public String toString() { // Print details of the sale in a formatted way // Print details of the shoe in a formatted way
            return String.format("ID: %d | %s - Size: %s - $%.2f - Qty: %d - Category: %s - Seller: %s - Added: %s",
                    id, name, size, price, quantity, category, seller, dateAdded.format(DATE_FORMAT));
        }
    }

    public static class SaleRecord extends Record { // Subclass representing a sale transaction
        final String buyerName; // Name of the buyer
        final double salePrice; // Price per item sold

        //Also inherits from Record, but represents a sale of a shoe.

        public SaleRecord(String name, String buyerName, int quantity, double salePrice) { // Constructor for SaleRecord
            super(name, quantity);
            this.buyerName = buyerName;
            this.salePrice = salePrice;
        }

        @Override
        //Prints sales history in a clean format showing quantity, price, buyer, and total.
        public String toString() {
            return String.format("ID: %d | %s - Buyer: %s - Qty: %d @ $%.2f - Total: $%.2f - Date: %s",
                    id, name, buyerName, quantity, salePrice, (quantity * salePrice),
                    dateAdded.format(DATE_TIME_FORMAT));
        }
    }


    public static void main(String[] args) { // Entry point of the application,Prompts user for name/email and saves them as a User object.
        Scanner input = new Scanner(System.in); // Read user input from console
        User currentUser = authenticateUser(input); // Prompt for name and email, create User object

        //Loads existing data from file store_data.ser, or starts empty if file not found.


        ArrayList<ShoeItem> inventory = loadInventory(); // Load inventory from file
        ArrayList<SaleRecord> sales = loadSales(); // Load sales data from file

        while (true) { // Main loop to display menu and handle commands
            displayMainMenu(currentUser); // Show the main menu
            String choice = input.nextLine().trim().toLowerCase(); // Read and normalize user input

            switch (choice) {
                case "1", "view" -> showInventory(inventory); // View all inventory items
                case "2", "add" -> addInventoryItem(inventory, input); // Add new shoe to inventory
                case "3", "delete" -> deleteInventoryItem(inventory, input); // Remove a shoe from inventory
                case "4", "sale" -> processSale(inventory, sales, input); // Record a sale
                case "5", "history" -> showSalesHistory(sales); // Display sales history
                case "6", "report" -> generateReports(inventory, sales, input); // Show reporting options
                case "7", "search" -> searchInventory(inventory, input); // Search inventory
                case "8", "exit" -> { // Exit the app and save data
                    saveData(inventory, sales); // Save inventory and sales to file
                    System.out.println("Goodbye, " + currentUser.username + "!"); // Farewell message
                    input.close(); // Close scanner
                    System.exit(0); // Exit the program
                }
                case "help" -> showHelp(); // Show available commands
                default -> System.out.println("Invalid option. Try again or type 'help'."); // Handle invalid command
            }
            System.out.println();
        }
    }

    private static User authenticateUser(Scanner input) {
        System.out.println("=== Unlace Shoe Inventory System ===");
        String username = getValidInput(input, "Enter your name: ", "Name cannot be empty.", s -> !s.isEmpty());
        String email = getValidInput(input, "Enter your email: ", "Invalid email format.",
                s -> s.matches("^[\\w-.]+@([\\w-]+\\.)+[\\w-]{2,4}$"));
        return new User(username, email);
    }

    private static void displayMainMenu(User user) {
        System.out.println("=== Welcome to Unlace, " + user.username + " ===");
        System.out.println("Session started: " + user.loginTime.format(DATE_TIME_FORMAT));
        System.out.println("\nMain Menu:");
        System.out.println("1. View Inventory (view)");
        System.out.println("2. Add New Shoe (add)");
        System.out.println("3. Delete Shoe (delete)");
        System.out.println("4. Record a Sale (sale)");
        System.out.println("5. View Sales History (history)");
        System.out.println("6. Generate Reports (report)");
        System.out.println("7. Search Inventory (search)");
        System.out.println("8. Exit (exit)");
        System.out.println("Type 'help' for command descriptions");
        System.out.print("Choose an option: ");
    }

    private static void showHelp() {
        System.out.println("\n=== Unlace Help ===");
        System.out.println("1/view   - Show current inventory");
        System.out.println("2/add    - Add new shoes to inventory");
        System.out.println("3/delete - Remove shoes from inventory");
        System.out.println("4/sale   - Record a shoe sale");
        System.out.println("5/history- View sales history");
        System.out.println("6/report - Generate inventory and sales reports");
        System.out.println("7/search - Search inventory by various criteria");
        System.out.println("8/exit   - Save data and exit the system");
        System.out.println("help     - Show this help message");
    }

    private static void generateReports(ArrayList<ShoeItem> inventory, ArrayList<SaleRecord> sales, Scanner input) {
        System.out.println("\n--- Reports ---");
        System.out.println("1. Inventory Summary");
        System.out.println("2. Sales Summary");
        System.out.println("3. Low Stock Alert");
        System.out.println("4. Best Selling Items");
        System.out.print("Choose report: ");

        String option = input.nextLine();
        switch (option) {
            case "1" -> generateInventoryReport(inventory);
            case "2" -> generateSalesReport(sales);
            case "3" -> generateLowStockReport(inventory);
            case "4" -> generateBestSellersReport(sales);
            default -> System.out.println("Invalid report option.");
        }
    }

    private static void cancelMessage() {
        System.out.println("Operation cancelled.");
    }

    // Inventory display, item addition, deletion, sale processing, sales history,
// searching, utility methods, input validation, data saving/loading methods follow:

    private static ArrayList<ShoeItem> loadInventory() { // Loads inventory list from file, or returns empty list if not found or error
        try (ObjectInputStream in = new ObjectInputStream(new FileInputStream(DATA_FILE))) {
            return (ArrayList<ShoeItem>) in.readObject();
        } catch (FileNotFoundException e) {
            System.out.println("No inventory data found. Starting fresh.");
            return new ArrayList<>();
        } catch (Exception e) {
            System.out.println("Error loading inventory: " + e.getMessage());
            return new ArrayList<>();
        }
    }

    private static ArrayList<SaleRecord> loadSales() { // Loads sales list from file, skipping inventory object
        try (ObjectInputStream in = new ObjectInputStream(new FileInputStream(DATA_FILE))) {
            in.readObject(); // skip inventory
            return (ArrayList<SaleRecord>) in.readObject();
        } catch (FileNotFoundException e) {
            System.out.println("No sales data found. Starting fresh.");
            return new ArrayList<>();
        } catch (Exception e) {
            System.out.println("Error loading sales: " + e.getMessage());
            return new ArrayList<>();
        }
    }

    private static String getValidInput(Scanner input, String prompt, String errorMessage, Predicate<String> validator) { // Repeatedly prompts until valid input is received based on the provided rule
        while (true) {
            System.out.print(prompt);
            String value = input.nextLine().trim();
            if (validator.test(value)) {
                return value;
            }
            System.out.println(errorMessage);
        }
    }

    private static int getValidInt(Scanner input, String prompt, int min, int max) { // Gets an integer within a valid range from user input
        while (true) {
            System.out.print(prompt);
            try {
                int value = Integer.parseInt(input.nextLine());
                if (value >= min && value <= max) return value;
            } catch (NumberFormatException ignored) {}
            System.out.printf("Enter a number between %d and %d.%n", min, max);
        }
    }

    private static double getValidDouble(Scanner input, String prompt, double min, double max) { // Gets a double within a valid range from user input
        while (true) {
            System.out.print(prompt);
            try {
                double value = Double.parseDouble(input.nextLine());
                if (value >= min && value <= max) return value;
            } catch (NumberFormatException ignored) {}
            System.out.printf("Enter a number between %.2f and %.2f.%n", min, max);
        }
    }
// --- Additional Methods ---

    private static void processSale(ArrayList<ShoeItem> inventory, ArrayList<SaleRecord> sales, Scanner input) { // Handles the process of selling a shoe and updating records
        if (inventory.isEmpty()) {
            System.out.println("Inventory is empty - nothing to sell.");
            return;
        }
        showInventory(inventory);
        int id = getValidInt(input, "Enter ID of shoe sold: ", 1, Integer.MAX_VALUE);
        Optional<ShoeItem> itemOpt = inventory.stream().filter(item -> item.id == id).findFirst();
        if (itemOpt.isEmpty()) {
            System.out.println("Item not found.");
            return;
        }
        ShoeItem item = itemOpt.get();
        String buyer = getValidInput(input, "Enter buyer's name: ", "Buyer name cannot be empty.", s -> !s.isEmpty());
        int qty = getValidInt(input, "Enter quantity sold: ", 1, item.quantity);
        double price = getValidDouble(input, "Enter sale price per item: ", 0.01, MAX_PRICE);
        item.quantity -= qty;
        sales.add(new SaleRecord(item.name, buyer, qty, price));
        System.out.println("Sale recorded.");
        if (item.quantity == 0) {
            inventory.remove(item);
            System.out.println("Item removed from inventory.");
        }
    }

    private static void showSalesHistory(ArrayList<SaleRecord> sales) { // Displays all recorded sales in history
        if (sales.isEmpty()) {
            System.out.println("No sales yet.");
            return;
        }
        System.out.println("--- Sales History ---");
        sales.forEach(System.out::println);
    }

    private static void searchInventory(ArrayList<ShoeItem> inventory, Scanner input) { // Allows user to search inventory items by name keyword
        if (inventory.isEmpty()) {
            System.out.println("Inventory is empty.");
            return;
        }
        //This line prompts the user to enter a search term and ensures that the input is not empty.
        String term = getValidInput(input, "Enter search term: ", "Cannot be empty.", s -> !s.isEmpty());
        //This segment filters the inventory list to find items whose names contain the search term, disregarding case sensitivity.​
        List<ShoeItem> results = inventory.stream()
                .filter(item -> item.name.toLowerCase().contains(term.toLowerCase()))
                .collect(Collectors.toList());
        if (results.isEmpty()) {
            System.out.println("No items matched.");
        } else {
            results.forEach(System.out::println);
        }
    }

    private static void generateInventoryReport(ArrayList<ShoeItem> inventory) { // Generates a summary of inventory including count and total value
        System.out.println("Inventory Summary");
        System.out.println("Total items: " + inventory.size());
        int totalQty = inventory.stream().mapToInt(item -> item.quantity).sum();
        double value = inventory.stream().mapToDouble(item -> item.quantity * item.price).sum();
        System.out.println("Total quantity: " + totalQty);
        System.out.printf("Total value: $%.2f%n", value);
    }

    private static void generateSalesReport(ArrayList<SaleRecord> sales) { // Generates total number of sales, items sold, and revenue
        System.out.println("Sales Summary");
        System.out.println("Total transactions: " + sales.size());
        int totalSold = sales.stream().mapToInt(s -> s.quantity).sum();
        double revenue = sales.stream().mapToDouble(s -> s.quantity * s.salePrice).sum();
        System.out.println("Total items sold: " + totalSold);
        System.out.printf("Total revenue: $%.2f%n", revenue);
    }

    private static void generateLowStockReport(ArrayList<ShoeItem> inventory) { // Shows shoes with stock less than 5 units
        System.out.println("Low Stock Items (<5):");
        inventory.stream()
                .filter(i -> i.quantity < 5)
                .forEach(i -> System.out.println(i.name + " (Qty: " + i.quantity + ")"));
    }

    private static void generateBestSellersReport(ArrayList<SaleRecord> sales) { // Shows top 5 best-selling shoe names by quantity sold
        System.out.println("Top 5 Best Sellers:");
        sales.stream()
                .collect(Collectors.groupingBy(s -> s.name, Collectors.summingInt(s -> s.quantity)))
                .entrySet().stream()
                .sorted((e1, e2) -> e2.getValue() - e1.getValue())
                .limit(5)
                .forEach(e -> System.out.println(e.getKey() + ": " + e.getValue() + " sold"));
    }

    private static void showInventory(ArrayList<ShoeItem> inventory) {
        if (inventory.isEmpty()) {
            System.out.println("Inventory is empty.");
            return;
        }
        System.out.println("--- Current Inventory ---");
        inventory.forEach(System.out::println);
    }

    private static void addInventoryItem(ArrayList<ShoeItem> inventory, Scanner input) {
        System.out.println("--- Add New Shoe ---");
        String name = getValidInput(input, "Enter shoe name: ", "Name cannot be empty.", s -> !s.isEmpty());
        double price = getValidDouble(input, "Enter price: ", 0.01, MAX_PRICE);
        int quantity = getValidInt(input, "Enter quantity: ", 1, MAX_QUANTITY);
        String seller = getValidInput(input, "Enter seller: ", "Seller cannot be empty.", s -> !s.isEmpty());
        String category = getValidInput(input, "Enter category: ", "Category cannot be empty.", s -> !s.isEmpty());
        String size = getValidInput(input, "Enter size: ", "Size cannot be empty.", s -> !s.isEmpty());

        ShoeItem item = new ShoeItem(name, price, quantity, seller, category, size);
        inventory.add(item);
        System.out.println("Shoe added: " + item);
    }

    private static void deleteInventoryItem(ArrayList<ShoeItem> inventory, Scanner input) {
        if (inventory.isEmpty()) {
            System.out.println("Inventory is empty.");
            return;
        }
        int id = getValidInt(input, "Enter shoe ID to delete: ", 1, Integer.MAX_VALUE);
        Optional<ShoeItem> match = inventory.stream().filter(i -> i.id == id).findFirst();
        if (match.isPresent()) {
            ShoeItem item = match.get();
            System.out.println("Found: " + item);
            String confirm = getValidInput(input, "Confirm deletion (yes/no): ", "Enter yes or no.", s -> s.equalsIgnoreCase("yes") || s.equalsIgnoreCase("no"));
            if (confirm.equalsIgnoreCase("yes")) {
                inventory.remove(item);
                System.out.println("Item deleted.");
            } else {
                cancelMessage();
            }
        } else {
            System.out.println("Item not found.");
        }
    }

    private static void saveData(ArrayList<ShoeItem> inventory, ArrayList<SaleRecord> sales) {
        try (ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream(DATA_FILE))) {
            out.writeObject(inventory);
            out.writeObject(sales);
            System.out.println("Data saved.");
        } catch (IOException e) {
            System.out.println("Save failed: " + e.getMessage());
        }
    }
}
