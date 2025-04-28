import org.junit.jupiter.api.*;

import java.io.ByteArrayOutputStream;
import java.io.PrintStream;
import java.time.LocalDate;

import static org.junit.jupiter.api.Assertions.*;

public class PoSCheckoutBadDesignTest {

    private final ByteArrayOutputStream output = new ByteArrayOutputStream();
    private final PrintStream originalOut = System.out;
    private PoSSystem pos;

    @BeforeEach
    void setup() {
        pos = new PoSSystem();
        System.setOut(new PrintStream(output));
        PoSSystem.clear();
    }

    @AfterEach
    void teardown() {
        System.setOut(originalOut);
    }

    private String getOutput() {
        return output.toString().trim();
    }

    @Test
    void testValidGeneralItemCheckout() {
        var customer = pos.new Customer(1, "Alice", "alice@example.com");
        PoSSystem.customers.add(customer);

        var item = pos.new Item("B101", "Notebook", 10.0, LocalDate.now().plusDays(3), "general");
        PoSSystem.stock.put(item.barcode, item);
        PoSSystem.inventory.put(item.barcode, 5);

        var sale = pos.new Sale(1, customer);
        sale.addItem(item, 2);

        sale.checkout(1, "CASH", 10.0, "USD");

        String out = getOutput();
        assertTrue(out.contains("Customer: Alice"));
        assertTrue(out.contains("Notebook x2 @ 10.00"));
        assertTrue(out.contains("Subtotal: $20.00"));
        assertTrue(out.contains("Discount Applied: -$2.00"));
        assertTrue(out.contains("Tax: +$1.60")); // 18 * 8%
        assertTrue(out.contains("Paid in cash"));
    }

    @Test
    void testCheckoutFailsWithExpiredItem() {
        var customer = pos.new Customer(2, "Bob", "bob@example.com");
        PoSSystem.customers.add(customer);

        var expiredItem = pos.new Item("B201", "Milk", 3.0, LocalDate.now().minusDays(1));
        PoSSystem.stock.put(expiredItem.barcode, expiredItem);
        PoSSystem.inventory.put(expiredItem.barcode, 2);

        var sale = pos.new Sale(2, customer);
        sale.addItem(expiredItem, 1);
        sale.checkout(2, "CARD", 0.0, "USD");

        assertTrue(getOutput().contains("Item not available or expired"));
    }

    @Test
    void testCheckoutFailsWithInsufficientStock() {
        var customer = pos.new Customer(3, "Cara", "cara@example.com");
        PoSSystem.customers.add(customer);

        var item = pos.new Item("B301", "Bread", 2.5, LocalDate.now().plusDays(2));
        PoSSystem.stock.put(item.barcode, item);
        PoSSystem.inventory.put(item.barcode, 1);

        var sale = pos.new Sale(3, customer);
        sale.addItem(item, 3); // over stock
        sale.checkout(3, "CASH", 0.0, "USD");

        assertTrue(getOutput().contains("Item not available or expired"));
    }

    @Test
    void testCheckoutFailsWithInvalidCurrency() {
        var customer = pos.new Customer(4, "Diana", "diana@example.com");
        PoSSystem.customers.add(customer);

        var item = pos.new Item("B401", "Juice", 5.0, LocalDate.now().plusDays(5));
        PoSSystem.stock.put(item.barcode, item);
        PoSSystem.inventory.put(item.barcode, 2);

        var sale = pos.new Sale(4, customer);
        sale.addItem(item, 1);
        sale.checkout(4, "CARD", 0.0, "INR");

        assertTrue(getOutput().contains("Unsupported currency"));
    }

    @Test
    void testCheckoutFailsWithUnsupportedPaymentMethod() {
        var customer = pos.new Customer(5, "Eric", "eric@example.com");
        PoSSystem.customers.add(customer);

        var item = pos.new Item("B501", "Pen", 1.0, LocalDate.now().plusDays(3));
        PoSSystem.stock.put(item.barcode, item);
        PoSSystem.inventory.put(item.barcode, 3);

        var sale = pos.new Sale(5, customer);
        sale.addItem(item, 1);
        sale.checkout(5, "BITCOIN", 0.0, "USD");

        assertTrue(getOutput().contains("Unsupported payment method"));
    }

    @Test
    void testCheckoutFailsWithDiscountOverLimit() {
        var customer = pos.new Customer(6, "Fiona", "fiona@example.com");
        PoSSystem.customers.add(customer);

        var item = pos.new Item("B601", "Towel", 15.0, LocalDate.now().plusDays(5));
        PoSSystem.stock.put(item.barcode, item);
        PoSSystem.inventory.put(item.barcode, 4);

        var sale = pos.new Sale(6, customer);
        sale.addItem(item, 1);
        sale.checkout(6, "CARD", 55.0, "USD");

        assertTrue(getOutput().contains("Discount too high!"));
    }

    @Test
    void testLuxuryItemAppliesHighTax() {
        var customer = pos.new Customer(7, "George", "george@example.com");
        PoSSystem.customers.add(customer);

        var item = pos.new Item("B701", "Watch", 100.0, LocalDate.now().plusDays(10), "luxury");
        PoSSystem.stock.put(item.barcode, item);
        PoSSystem.inventory.put(item.barcode, 2);

        var sale = pos.new Sale(7, customer);
        sale.addItem(item, 1);
        sale.checkout(7, "CARD", 0.0, "USD");

        assertTrue(getOutput().contains("Tax: +$20.00"));
    }

    @Test
    void testMedicineItemIsTaxFree() {
        var customer = pos.new Customer(8, "Hannah", "hannah@example.com");
        PoSSystem.customers.add(customer);

        var item = pos.new Item("B801", "Antibiotic", 25.0, LocalDate.now().plusDays(10), "medicine");
        PoSSystem.stock.put(item.barcode, item);
        PoSSystem.inventory.put(item.barcode, 1);

        var sale = pos.new Sale(8, customer);
        sale.addItem(item, 1);
        sale.checkout(8, "CASH", 0.0, "USD");

        assertTrue(getOutput().contains("Tax: +$0.00"));
    }
}
