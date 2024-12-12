# TugasPBODatabase Fadhilla Firma
import java.sql.*;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.Scanner;

// Custom exception untuk login gagal
class LoginFailedException extends Exception {
    public LoginFailedException(String message) {
        super(message);
    }
}

// Custom exception untuk input transaksi
class InvalidTransactionException extends Exception {
    public InvalidTransactionException(String message) {
        super(message);
    }
}

// Parent class untuk menangani login
class LoginSystem {
    private final String USERNAME = "kasir";
    private final String PASSWORD = "qwerty";

    public void login(String inputUsername, String inputPassword) throws LoginFailedException {
        if (!inputUsername.equals(USERNAME) || !inputPassword.equals(PASSWORD)) {
            throw new LoginFailedException("Login gagal, silakan coba lagi.");
        }
    }
}

// Class untuk JDBC dan operasi CRUD
class TransactionDatabase {
    private final String DB_URL = "jdbc:mysql://localhost:3306/SupermarketDB";
    private final String DB_USER = "root";
    private final String DB_PASSWORD = ""; // Sesuaikan password Anda

    public Connection connect() throws SQLException {
        return DriverManager.getConnection(DB_URL, DB_USER, DB_PASSWORD);
    }

    public void insertTransaction(String noFaktur, String kodeBarang, String namaBarang, double hargaBarang,
            int jumlahBeli, double total, String namaKasir) {
        String query = "INSERT INTO Transaksi (noFaktur, kodeBarang, namaBarang, hargaBarang, jumlahBeli, total, namaKasir, tanggalTransaksi) VALUES (?, ?, ?, ?, ?, ?, ?, ?)";
        try (Connection conn = connect(); PreparedStatement stmt = conn.prepareStatement(query)) {
            stmt.setString(1, noFaktur);
            stmt.setString(2, kodeBarang);
            stmt.setString(3, namaBarang);
            stmt.setDouble(4, hargaBarang);
            stmt.setInt(5, jumlahBeli);
            stmt.setDouble(6, total);
            stmt.setString(7, namaKasir);
            stmt.setTimestamp(8, new Timestamp(System.currentTimeMillis()));
            stmt.executeUpdate();
            System.out.println("Data transaksi berhasil disimpan ke database.");
        } catch (SQLException e) {
            System.out.println("Kesalahan saat menyimpan data: " + e.getMessage());
        }
    }

    public void displayTransactions() {
        String query = "SELECT * FROM Transaksi";
        try (Connection conn = connect();
                Statement stmt = conn.createStatement();
                ResultSet rs = stmt.executeQuery(query)) {
            System.out.println("\nData Transaksi:");
            while (rs.next()) {
                System.out.println("ID: " + rs.getInt("id") + ", Faktur: " + rs.getString("noFaktur") + ", Nama Barang: "
                        + rs.getString("namaBarang") + ", Total: " + rs.getDouble("total") + ", Kasir: "
                        + rs.getString("namaKasir") + ", Tanggal: " + rs.getTimestamp("tanggalTransaksi"));
            }
        } catch (SQLException e) {
            System.out.println("Kesalahan saat membaca data: " + e.getMessage());
        }
    }

    public void deleteTransaction(int id) {
        String query = "DELETE FROM Transaksi WHERE id = ?";
        try (Connection conn = connect(); PreparedStatement stmt = conn.prepareStatement(query)) {
            stmt.setInt(1, id);
            int rowsDeleted = stmt.executeUpdate();
            if (rowsDeleted > 0) {
                System.out.println("Data transaksi berhasil dihapus.");
            } else {
                System.out.println("Data transaksi dengan ID tersebut tidak ditemukan.");
            }
        } catch (SQLException e) {
            System.out.println("Kesalahan saat menghapus data: " + e.getMessage());
        }
    }

    public void updateTransaction(int id, String noFaktur, String kodeBarang, String namaBarang, double hargaBarang,
            int jumlahBeli, double total, String namaKasir) {
        String query = "UPDATE Transaksi SET noFaktur = ?, kodeBarang = ?, namaBarang = ?, hargaBarang = ?, jumlahBeli = ?, total = ?, namaKasir = ? WHERE id = ?";
        try (Connection conn = connect(); PreparedStatement stmt = conn.prepareStatement(query)) {
            stmt.setString(1, noFaktur);
            stmt.setString(2, kodeBarang);
            stmt.setString(3, namaBarang);
            stmt.setDouble(4, hargaBarang);
            stmt.setInt(5, jumlahBeli);
            stmt.setDouble(6, total);
            stmt.setString(7, namaKasir);
            stmt.setInt(8, id);
            int rowsUpdated = stmt.executeUpdate();
            if (rowsUpdated > 0) {
                System.out.println("Data transaksi berhasil diperbarui.");
            } else {
                System.out.println("Data transaksi dengan ID tersebut tidak ditemukan.");
            }
        } catch (SQLException e) {
            System.out.println("Kesalahan saat mengupdate data: " + e.getMessage());
        }
    }
}

// Child class untuk transaksi
class TransactionSystem extends LoginSystem {
    private final TransactionDatabase db = new TransactionDatabase();

    public void processTransaction() {
        Scanner scanner = new Scanner(System.in);

        try {
            // Menampilkan tanggal dan waktu
            Date date = new Date();
            SimpleDateFormat dateFormatter = new SimpleDateFormat("MM/dd/yy");
            SimpleDateFormat timeFormatter = new SimpleDateFormat("HH:mm:ss");

            System.out.println("+-----------------------------------------------------+");
            System.out.println("Selamat Datang di Supermarket Fadimart");
            System.out.println("Tanggal dan Waktu : " + dateFormatter.format(date) + " " + timeFormatter.format(date));
            System.out.println("+-----------------------------------------------------+");

            // Input Nama Kasir
            System.out.print("Nama Kasir: ");
            String namaKasir = scanner.nextLine();

            // Input No Faktur
            System.out.print("No. Faktur: ");
            String noFaktur = scanner.nextLine();
            if (noFaktur.isEmpty()) {
                throw new InvalidTransactionException("No. Faktur tidak boleh kosong!");
            }

            // Input Kode Barang dan detail lainnya
            System.out.print("Kode Barang: ");
            String kodeBarang = scanner.nextLine();

            System.out.print("Nama Barang: ");
            String namaBarang = scanner.nextLine();

            System.out.print("Harga Barang: ");
            double hargaBarang = scanner.nextDouble();

            System.out.print("Jumlah Beli: ");
            int jumlahBeli = scanner.nextInt();
            if (jumlahBeli <= 0) {
                throw new InvalidTransactionException("Jumlah beli tidak boleh kurang dari atau sama dengan 0.");
            }

            // Perhitungan total
            double total = hargaBarang * jumlahBeli;

            // Menampilkan hasil transaksi
            System.out.println("+-----------------------------------------------------+");
            System.out.println("No. Faktur      : " + noFaktur);
            System.out.println("Kode Barang     : " + kodeBarang);
            System.out.println("Nama Barang     : " + namaBarang);
            System.out.println("Harga Barang    : Rp " + hargaBarang);
            System.out.println("Jumlah Beli     : " + jumlahBeli);
            System.out.println("TOTAL           : Rp " + total);
            System.out.println("+-----------------------------------------------------+");
            System.out.println("Kasir           : " + namaKasir);
            System.out.println("+-----------------------------------------------------+");

            // Simpan transaksi ke database
            db.insertTransaction(noFaktur, kodeBarang, namaBarang, hargaBarang, jumlahBeli, total, namaKasir);

        } catch (InvalidTransactionException e) {
            System.out.println("Kesalahan: " + e.getMessage());
        } catch (Exception e) {
            System.out.println("Terjadi kesalahan input. Silakan ulangi!");
        }
    }
}

// Main class
public class Maindb {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        TransactionSystem transactionSystem = new TransactionSystem();
        TransactionDatabase db = new TransactionDatabase();

        // Login process
        try {
            System.out.println("Log in");
            System.out.println("+-----------------------------------------------------+");
            System.out.print("Username: ");
            String inputUsername = scanner.nextLine();
            System.out.print("Password: ");
            String inputPassword = scanner.nextLine();

            // Validasi login
            transactionSystem.login(inputUsername, inputPassword);
            System.out.println("Login berhasil!");

            // Menu utama
            boolean running = true;
            while (running) {
                System.out.println("\nMenu:");
                System.out.println("1. Input Transaksi");
                System.out.println("2. Tampilkan Data Transaksi");
                System.out.println("3. Hapus Data Transaksi");
                System.out.println("4. Update Data Transaksi");
                System.out.println("5. Keluar");
                System.out.print("Pilih opsi: ");
                int choice = scanner.nextInt();

                switch (choice) {
                    case 1:
                        transactionSystem.processTransaction();
                        break;
                    case 2:
                        db.displayTransactions();
                        break;
                    case 3:
                        System.out.print("Masukkan ID transaksi yang ingin dihapus: ");
                        int idDelete = scanner.nextInt();
                        db.deleteTransaction(idDelete);
                        break;
                    case 4:
                        // Update transaksi
                        System.out.print("Masukkan ID transaksi yang ingin diperbarui: ");
                        int idUpdate = scanner.nextInt();
                        scanner.nextLine(); // Consume newline character

                        // Input detail transaksi yang baru
                        System.out.print("No. Faktur: ");
                        String noFaktur = scanner.nextLine();
                        System.out.print("Kode Barang: ");
                        String kodeBarang = scanner.nextLine();
                        System.out.print("Nama Barang: ");
                        String namaBarang = scanner.nextLine();
                        System.out.print("Harga Barang: ");
                        double hargaBarang = scanner.nextDouble();
                        System.out.print("Jumlah Beli: ");
                        int jumlahBeli = scanner.nextInt();
                        double total = hargaBarang * jumlahBeli;

                        // Input nama kasir
                        scanner.nextLine();  // Consume newline
                        System.out.print("Nama Kasir: ");
                        String namaKasir = scanner.nextLine();

                        db.updateTransaction(idUpdate, noFaktur, kodeBarang, namaBarang, hargaBarang, jumlahBeli, total, namaKasir);
                        break;
                    case 5:
                        running = false;
                        System.out.println("Keluar dari program...");
                        break;
                    default:
                        System.out.println("Pilihan tidak valid.");
                        break;
                }
            }
        } catch (LoginFailedException e) {
            System.out.println(e.getMessage());
        }
    }
}
