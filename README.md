# program_pengelolaan_karya_ilmiah_Nim_181

#include <iostream>
#include <vector>
#include <string>
#include <fstream>
#include <sstream>
#include <limits>
#include <algorithm>
#include <cctype>
#include <map>

// Fungsi untuk membersihkan layar
void clearScreen() {
#ifdef _WIN32
    system("cls"); // Untuk Windows
#else
    system("clear"); // Untuk Unix/Linux/Mac
#endif
}

// Struktur untuk menyimpan data hasil karya ilmiah
struct KaryaIlmiah {
    int id;
    std::string judul;
    std::string penulis;
    int tahun;
    std::string jenis;
};
// Deklarasi fungsi yang digunakan
void cariKaryaIlmiah(const std::vector<KaryaIlmiah> &database);
void hapusKaryaIlmiah(std::vector<KaryaIlmiah> &database);
void editKaryaIlmiah(std::vector<KaryaIlmiah> &database);
void sortirKaryaIlmiah(std::vector<KaryaIlmiah> &database);

// Fungsi untuk memuat data dari file ke dalam vector
void loadData(std::vector<KaryaIlmiah> &database, const std::string &filename) {
    std::ifstream file(filename);
    if (!file.is_open()) {
        // Jika file tidak ada, buat file baru
        std::ofstream outfile(filename);
        outfile.close();
        return;
    }

    std::string line;
    while (std::getline(file, line)) {
        if (line.empty()) continue; // Lewati baris kosong
        std::stringstream ss(line);
        KaryaIlmiah karya;
        std::string token;

        // Parsing ID
        std::getline(ss, token, ',');
        karya.id = std::stoi(token);

        // Parsing Judul
        std::getline(ss, token, ',');
        karya.judul = token;

        // Parsing Penulis
        std::getline(ss, token, ',');
        karya.penulis = token;

        // Parsing Tahun
        std::getline(ss, token, ',');
        karya.tahun = std::stoi(token);

        // Parsing Jenis
        std::getline(ss, token, ',');
        karya.jenis = token;

        database.push_back(karya);
    }

    file.close();
}

// Fungsi untuk menyimpan data dari vector ke file
void saveData(const std::vector<KaryaIlmiah> &database, const std::string &filename) {
    std::ofstream file(filename, std::ios::trunc); // Buka file untuk ditulis ulang
    if (!file.is_open()) {
        std::cerr << "Gagal membuka file untuk menyimpan data.\n";
        return;
    }

    for (const auto &karya : database) {
        file << karya.id << ","
             << karya.judul << ","
             << karya.penulis << ","
             << karya.tahun << ","
             << karya.jenis << "\n";
    }

    file.close();
}

// Fungsi untuk mendapatkan ID unik
int generateID(const std::vector<KaryaIlmiah> &database) {
    int maxID = 0;
    for (const auto &karya : database) {
        if (karya.id > maxID)
            maxID = karya.id;
    }
    return maxID + 1;
}

// Fungsi untuk menambahkan data karya ilmiah baru
void tambahKaryaIlmiah(std::vector<KaryaIlmiah> &database) {
    clearScreen(); // Bersihkan layar
    KaryaIlmiah karya;
    karya.id = generateID(database);
    std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n'); // Membersihkan buffer

    std::cout << "\n-- Tambah Data Hasil Karya Ilmiah --\n";
    std::cout << "Judul             : ";
    std::getline(std::cin, karya.judul);
    std::cout << "Penulis           : ";
    std::getline(std::cin, karya.penulis);

    bool validTahun = false;
    while (!validTahun) {
        std::cout << "Tahun Terbit      : ";
        std::cin >> karya.tahun;
        if (std::cin.fail() || karya.tahun < 1900 || karya.tahun > 2100) {
            std::cin.clear();
            std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
            std::cout << "Tahun tidak valid. Masukkan angka antara 1900 - 2100.\n";
        } else {
            validTahun = true;
        }
    }
    std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n'); // Membersihkan buffer

    std::cout << "Jenis Karya (Tesis/Jurnal/Makalah/Konferensi, dll.): ";
    std::getline(std::cin, karya.jenis);

    database.push_back(karya);
    std::cout << "\nData berhasil ditambahkan dengan ID: " << karya.id << "\n\n";
}

void tampilkanData(std::vector<KaryaIlmiah> &database) {
    clearScreen(); // Bersihkan layar
    if (database.empty()) {
        std::cout << "\nBelum ada data hasil karya ilmiah yang tersimpan.\n\n";
        return;
    }

    const int barisPerHalaman = 5;
    int halaman = 0;
    bool selesai = false;

    while (!selesai) {
        clearScreen(); // Bersihkan layar setiap kali tampil ulang
        int start = halaman * barisPerHalaman;
        int end = std::min(start + barisPerHalaman, static_cast<int>(database.size()));

        std::cout << "\n-- Daftar Hasil Karya Ilmiah (Halaman " << (halaman + 1) << ") --\n";
        for (int i = start; i < end; ++i) {
            const auto &karya = database[i];
            std::cout << "ID     : " << karya.id << "\n";
            std::cout << "Judul  : " << karya.judul << "\n";
            std::cout << "Penulis: " << karya.penulis << "\n";
            std::cout << "Tahun  : " << karya.tahun << "\n";
            std::cout << "Jenis  : " << karya.jenis << "\n";
            std::cout << "-----------------------------\n";
        }

        std::cout << "Jumlah data: " << database.size() << "\n";
        std::cout << "Data per halaman: " << barisPerHalaman << "\n";

        // Menu interaktif
        std::cout << "\nMenu: [N]ext, [P]revious, [C]ari, [H]apus, [E]dit, [S]ortir, [Q]uit: ";
        char pilihan;
        std::cin >> pilihan;

        pilihan = std::tolower(pilihan); // Ubah input ke huruf kecil untuk konsistensi

        switch (pilihan) {
            case 'n': // Next page
                if (end < database.size()) {
                    ++halaman;
                } else {
                    std::cout << "Ini adalah halaman terakhir.\n";
                    std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
                    std::cin.get(); // Tunggu input untuk lanjut
                }
                break;
            case 'p': // Previous page
                if (halaman > 0) {
                    --halaman;
                } else {
                    std::cout << "Ini adalah halaman pertama.\n";
                    std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
                    std::cin.get(); // Tunggu input untuk lanjut
                }
                break;
            case 'c': // Cari data
                cariKaryaIlmiah(database);
                selesai = true; // Keluar dari tampilan untuk fungsi pencarian
                break;
            case 'h': // Hapus data
                hapusKaryaIlmiah(database);
                saveData(database, "database.txt"); // Simpan setelah penghapusan
                selesai = true; // Keluar untuk menyesuaikan data
                break;
            case 'e': // Edit data
                editKaryaIlmiah(database);
                saveData(database, "database.txt"); // Simpan setelah pengeditan
                selesai = true; // Keluar untuk menyesuaikan data
                break;
            case 's': // Sortir data
                sortirKaryaIlmiah(database);
                selesai = true; // Keluar untuk menampilkan ulang data setelah penyortiran
                break;
            case 'q': // Quit menu
                selesai = true;
                break;
            default:
                std::cout << "Input tidak valid. Silakan coba lagi.\n";
                std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
                std::cin.get(); // Tunggu input untuk lanjut
                break;
        }
    }
}


// Fungsi untuk mencari data karya ilmiah berdasarkan judul atau penulis dengan pagination
void cariKaryaIlmiah(const std::vector<KaryaIlmiah> &database) {
    clearScreen(); // Bersihkan layar
    if (database.empty()) {
        std::cout << "\nTidak ada data untuk dicari.\n\n";
        return;
    }

    std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
    std::string keyword;
    std::cout << "\nMasukkan Judul atau Penulis yang dicari: ";
    std::getline(std::cin, keyword);

    std::vector<KaryaIlmiah> hasilPencarian;
    for (const auto &karya : database) {
        std::string judulLower = karya.judul;
        std::string penulisLower = karya.penulis;
        std::string keywordLower = keyword;

        std::transform(judulLower.begin(), judulLower.end(), judulLower.begin(), ::tolower);
        std::transform(penulisLower.begin(), penulisLower.end(), penulisLower.begin(), ::tolower);
        std::transform(keywordLower.begin(), keywordLower.end(), keywordLower.begin(), ::tolower);

        if (judulLower.find(keywordLower) != std::string::npos ||
            penulisLower.find(keywordLower) != std::string::npos) {
            hasilPencarian.push_back(karya);
        }
    }

    if (hasilPencarian.empty()) {
        std::cout << "Tidak ditemukan hasil karya ilmiah yang sesuai dengan pencarian.\n\n";
        return;
    }

    tampilkanData(hasilPencarian);
}

void hapusKaryaIlmiah(std::vector<KaryaIlmiah> &database) {
    clearScreen(); // Bersihkan layar
    if (database.empty()) {
        std::cout << "\nBelum ada data hasil karya ilmiah untuk dihapus.\n\n";
        return;
    }

    const int barisPerHalaman = 5;
    int halaman = 0;
    bool selesai = false;

    while (!selesai) {
        clearScreen(); // Bersihkan layar setiap kali tampil ulang
        int start = halaman * barisPerHalaman;
        int end = std::min(start + barisPerHalaman, static_cast<int>(database.size()));

        std::cout << "\n-- Hapus Data Hasil Karya Ilmiah (Halaman " << (halaman + 1) << ") --\n";
        for (int i = start; i < end; ++i) {
            const auto &karya = database[i];
            std::cout << "ID     : " << karya.id << "\n";
            std::cout << "Judul  : " << karya.judul << "\n";
            std::cout << "Penulis: " << karya.penulis << "\n";
            std::cout << "Tahun  : " << karya.tahun << "\n";
            std::cout << "Jenis  : " << karya.jenis << "\n";
            std::cout << "-----------------------------\n";
        }

        std::cout << "Jumlah data: " << database.size() << "\n";
        std::cout << "Data per halaman: " << barisPerHalaman << "\n";

        // Menu interaktif
        std::cout << "\nMenu: [N]ext, [P]revious, [I]nput ID untuk Hapus, [Q]uit: ";
        char pilihan;
        std::cin >> pilihan;

        pilihan = std::tolower(pilihan); // Ubah input ke huruf kecil untuk konsistensi

        switch (pilihan) {
            case 'n': // Next page
                if (end < database.size()) {
                    ++halaman;
                } else {
                    std::cout << "Ini adalah halaman terakhir.\n";
                    std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
                    std::cin.get(); // Tunggu input untuk lanjut
                }
                break;
            case 'p': // Previous page
                if (halaman > 0) {
                    --halaman;
                } else {
                    std::cout << "Ini adalah halaman pertama.\n";
                    std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
                    std::cin.get(); // Tunggu input untuk lanjut
                }
                break;
            case 'i': { // Input ID untuk hapus
                int idHapus;
                std::cout << "\nMasukkan ID karya ilmiah yang akan dihapus: ";
                std::cin >> idHapus;

                if (std::cin.fail()) {
                    std::cin.clear();
                    std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
                    std::cout << "Input tidak valid. ID harus berupa angka.\n\n";
                } else {
                    auto it = std::find_if(database.begin(), database.end(), [idHapus](const KaryaIlmiah &karya) {
                        return karya.id == idHapus;
                    });

                    if (it != database.end()) {
                        database.erase(it);
                        std::cout << "Data dengan ID " << idHapus << " berhasil dihapus.\n\n";
                        saveData(database, "database.txt"); // Simpan setelah penghapusan
                        selesai = true; // Keluar setelah penghapusan
                    } else {
                        std::cout << "Tidak ditemukan data dengan ID " << idHapus << ".\n\n";
                        std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
                        std::cin.get(); // Tunggu input untuk lanjut
                    }
                }
                break;
            }
            case 'q': // Quit menu
                selesai = true;
                break;
            default:
                std::cout << "Input tidak valid. Silakan coba lagi.\n";
                std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
                std::cin.get(); // Tunggu input untuk lanjut
                break;
        }
    }
}

// Fungsi untuk mengedit data karya ilmiah
void editKaryaIlmiah(std::vector<KaryaIlmiah> &database) {
    clearScreen(); // Bersihkan layar
    if (database.empty()) {
        std::cout << "\nBelum ada data hasil karya ilmiah untuk diedit.\n\n";
        return;
    }

    const int barisPerHalaman = 5;
    int halaman = 0;
    bool selesai = false;

    while (!selesai) {
        clearScreen(); // Bersihkan layar setiap kali tampil ulang
        int start = halaman * barisPerHalaman;
        int end = std::min(start + barisPerHalaman, static_cast<int>(database.size()));

        std::cout << "\n-- Edit Data Hasil Karya Ilmiah (Halaman " << (halaman + 1) << ") --\n";
        for (int i = start; i < end; ++i) {
            const auto &karya = database[i];
            std::cout << "ID     : " << karya.id << "\n";
            std::cout << "Judul  : " << karya.judul << "\n";
            std::cout << "Penulis: " << karya.penulis << "\n";
            std::cout << "Tahun  : " << karya.tahun << "\n";
            std::cout << "Jenis  : " << karya.jenis << "\n";
            std::cout << "-----------------------------\n";
        }

        std::cout << "Jumlah data: " << database.size() << "\n";
        std::cout << "Data per halaman: " << barisPerHalaman << "\n";

        // Menu interaktif
        std::cout << "\nMenu: [N]ext, [P]revious, [I]nput ID untuk Edit, [Q]uit: ";
        char pilihan;
        std::cin >> pilihan;

        pilihan = std::tolower(pilihan); // Ubah input ke huruf kecil untuk konsistensi

        switch (pilihan) {
            case 'n': // Next page
                if (end < database.size()) {
                    ++halaman;
                } else {
                    std::cout << "Ini adalah halaman terakhir.\n";
                    std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
                    std::cin.get(); // Tunggu input untuk lanjut
                }
                break;
            case 'p': // Previous page
                if (halaman > 0) {
                    --halaman;
                } else {
                    std::cout << "Ini adalah halaman pertama.\n";
                    std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
                    std::cin.get(); // Tunggu input untuk lanjut
                }
                break;
            case 'i': { // Input ID untuk edit
                int idEdit;
                std::cout << "\nMasukkan ID karya ilmiah yang akan diedit: ";
                std::cin >> idEdit;

                if (std::cin.fail()) {
                    std::cin.clear();
                    std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
                    std::cout << "Input tidak valid. ID harus berupa angka.\n\n";
                } else {
                    auto it = std::find_if(database.begin(), database.end(), [idEdit](const KaryaIlmiah &karya) {
                        return karya.id == idEdit;
                    });

                    if (it != database.end()) {
                        KaryaIlmiah &karya = *it;
                        std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n'); // Bersihkan buffer

                        std::cout << "\n-- Edit Data Hasil Karya Ilmiah --\n";
                        std::cout << "Judul saat ini: " << karya.judul << "\n";
                        std::cout << "Judul baru (kosongkan jika tidak ingin mengubah): ";
                        std::string input;
                        std::getline(std::cin, input);
                        if (!input.empty()) karya.judul = input;

                        std::cout << "Penulis saat ini: " << karya.penulis << "\n";
                        std::cout << "Penulis baru (kosongkan jika tidak ingin mengubah): ";
                        std::getline(std::cin, input);
                        if (!input.empty()) karya.penulis = input;

                        std::cout << "Tahun saat ini: " << karya.tahun << "\n";
                        std::cout << "Tahun baru (0 jika tidak ingin mengubah): ";
                        int tahunBaru;
                        std::cin >> tahunBaru;
                        if (!std::cin.fail() && tahunBaru != 0) karya.tahun = tahunBaru;
                        std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');

                        std::cout << "Jenis saat ini: " << karya.jenis << "\n";
                        std::cout << "Jenis baru (kosongkan jika tidak ingin mengubah): ";
                        std::getline(std::cin, input);
                        if (!input.empty()) karya.jenis = input;

                        std::cout << "\nData berhasil diperbarui.\n\n";
                        saveData(database, "database.txt"); // Simpan setelah pengeditan
                        selesai = true; // Keluar setelah pengeditan
                    } else {
                        std::cout << "Tidak ditemukan data dengan ID " << idEdit << ".\n\n";
                        std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
                        std::cin.get(); // Tunggu input untuk lanjut
                    }
                }
                break;
            }
            case 'q': // Quit menu
                selesai = true;
                break;
            default:
                std::cout << "Input tidak valid. Silakan coba lagi.\n";
                std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
                std::cin.get(); // Tunggu input untuk lanjut
                break;
        }
    }
}

// Fungsi untuk menyortir data karya ilmiah
void sortirKaryaIlmiah(std::vector<KaryaIlmiah> &database) {
    clearScreen(); // Bersihkan layar
    if (database.empty()) {
        std::cout << "\nBelum ada data hasil karya ilmiah untuk disortir.\n\n";
        return;
    }

    std::cout << "\n-- Pilih Kriteria Sortir --\n";
    std::cout << "1. Berdasarkan Judul\n";
    std::cout << "2. Berdasarkan Penulis\n";
    std::cout << "3. Berdasarkan Tahun\n";
    std::cout << "4. Berdasarkan Jenis\n";
    std::cout << "Pilih (1-4): ";

    int pilihan;
    std::cin >> pilihan;

    switch (pilihan) {
        case 1:
            std::sort(database.begin(), database.end(), [](const KaryaIlmiah &a, const KaryaIlmiah &b) {
                return a.judul < b.judul;
            });
            std::cout << "\nData berhasil disortir berdasarkan Judul.\n\n";
            break;
        case 2:
            std::sort(database.begin(), database.end(), [](const KaryaIlmiah &a, const KaryaIlmiah &b) {
                return a.penulis < b.penulis;
            });
            std::cout << "\nData berhasil disortir berdasarkan Penulis.\n\n";
            break;
        case 3:
            std::sort(database.begin(), database.end(), [](const KaryaIlmiah &a, const KaryaIlmiah &b) {
                return a.tahun < b.tahun;
            });
            std::cout << "\nData berhasil disortir berdasarkan Tahun.\n\n";
            break;
        case 4:
            std::sort(database.begin(), database.end(), [](const KaryaIlmiah &a, const KaryaIlmiah &b) {
                return a.jenis < b.jenis;
            });
            std::cout << "\nData berhasil disortir berdasarkan Jenis.\n\n";
            break;
        default:
            std::cout << "Pilihan tidak valid.\n\n";
            break;
    }
}

void tampilkanStatistik(const std::vector<KaryaIlmiah> &database) {
    clearScreen(); // Bersihkan layar
    if (database.empty()) {
        std::cout << "\nTidak ada data untuk ditampilkan statistiknya.\n\n";
        return;
    }

    std::cout << "\n=== Statistik Data Karya Ilmiah ===\n";

    // Hitung jumlah berdasarkan jenis
    std::map<std::string, int> jenisCount;
    for (const auto &karya : database) {
        jenisCount[karya.jenis]++;
    }

    std::cout << "\nJumlah Berdasarkan Jenis:\n";
    for (const auto &entry : jenisCount) {
        std::cout << "- " << entry.first << ": " << entry.second << "\n";
    }

    // Hitung jumlah berdasarkan tahun
    std::map<int, int> tahunCount;
    for (const auto &karya : database) {
        tahunCount[karya.tahun]++;
    }

    std::cout << "\nJumlah Berdasarkan Tahun:\n";
    for (const auto &entry : tahunCount) {
        std::cout << "- " << entry.first << ": " << entry.second << "\n";
    }

    std::cout << "\nTekan enter untuk kembali ke menu utama.";
    std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
    std::cin.get(); // Tunggu input untuk lanjut
}

void eksporDataCSV(const std::vector<KaryaIlmiah> &database, const std::string &filename) {
    clearScreen(); // Bersihkan layar
    if (database.empty()) {
        std::cout << "\nTidak ada data untuk diekspor.\n\n";
        return;
    }

    std::ofstream file(filename);
    if (!file.is_open()) {
        std::cerr << "\nGagal membuka file untuk ekspor.\n\n";
        return;
    }

    // Tulis header
    file << "ID,Judul,Penulis,Tahun,Jenis\n";

    // Tulis data
    for (const auto &karya : database) {
        file << karya.id << ","
             << karya.judul << ","
             << karya.penulis << ","
             << karya.tahun << ","
             << karya.jenis << "\n";
    }

    file.close();
    std::cout << "\nData berhasil diekspor ke " << filename << ".\n\n";
    std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
    std::cin.get(); // Tunggu input untuk lanjut
}

// Fungsi untuk menampilkan statistik karya ilmiah
void tampilkanStatistikTren(const std::vector<KaryaIlmiah> &database) {
    std::map<int, int> tahunCount;   // Untuk menghitung karya ilmiah per tahun
    std::map<std::string, int> jenisCount;  // Untuk menghitung karya ilmiah per jenis
    std::map<int, int> tahunIncrease; // Untuk menghitung peningkatan per tahun

    // Menghitung jumlah karya ilmiah per tahun dan jenis
    for (const auto& karya : database) {
        tahunCount[karya.tahun]++;
        jenisCount[karya.jenis]++;
    }

    // Menampilkan jumlah karya ilmiah per tahun
    std::cout << "\nJumlah Karya Ilmiah Per Tahun:\n";
    for (const auto& entry : tahunCount) {
        std::cout << "Tahun " << entry.first << ": " << entry.second << " karya ilmiah\n";
    }

    // Menampilkan jumlah karya ilmiah per jenis
    std::cout << "\nJumlah Karya Ilmiah Per Jenis:\n";
    for (const auto& entry : jenisCount) {
        std::cout << entry.first << ": " << entry.second << " karya ilmiah\n";
    }

    // Menghitung peningkatan jumlah karya ilmiah dari tahun ke tahun
    for (const auto& entry : tahunCount) {
        int tahun = entry.first;
        int jumlah = entry.second;
        if (tahunCount.find(tahun - 1) != tahunCount.end()) {
            tahunIncrease[tahun] = jumlah - tahunCount[tahun - 1];
        }
    }

    // Menampilkan peningkatan karya ilmiah per tahun
    std::cout << "\nPeningkatan Karya Ilmiah Per Tahun:\n";
    for (const auto& entry : tahunIncrease) {
        std::cout << "Tahun " << entry.first << ": " << entry.second << " peningkatan karya ilmiah\n";
    }
}

int main() {
    std::vector<KaryaIlmiah> database;
    const std::string filename = "database.txt";

    // Memuat data dari file saat program dimulai
    loadData(database, filename);

    bool running = true;

    while (running) {
        std::cout << "=== PROGRAM PENGELOLAAN HASIL KARYA ILMIAH PERPUSTAKAAN ===\n";
        std::cout << "1. Tambah Data Hasil Karya Ilmiah\n";
        std::cout << "2. Tampilkan Semua Data Hasil Karya Ilmiah\n";
        std::cout << "3. Cari Data Hasil Karya Ilmiah\n";
        std::cout << "4. Hapus Data Hasil Karya Ilmiah\n";
        std::cout << "5. Edit Data Hasil Karya Ilmiah\n";
        std::cout << "6. Sortir Data Hasil Karya Ilmiah\n";
        std::cout << "7. Tampilkan Statistik Data Karya Ilmiah\n";
        std::cout << "8. Tampilkan Statistik Tren Karya Ilmiah\n";
        std::cout << "9. Ekspor Data ke CSV\n";
        std::cout << "10. Keluar Program\n";
        std::cout << "Pilih menu (1-10): ";

        int pilihan;
        std::cin >> pilihan;

        if (std::cin.fail()) {
            std::cin.clear(); 
            std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
            std::cout << "\nInput tidak valid, silakan coba lagi.\n\n";
            continue;
        }

        switch (pilihan) {
            case 1:
                tambahKaryaIlmiah(database);
                saveData(database, filename); // Simpan data setelah penambahan
                break;
            case 2:
                tampilkanData(database);
                break;
            case 3:
                cariKaryaIlmiah(database);
                break;
            case 4:
                hapusKaryaIlmiah(database);
                saveData(database, filename); // Simpan data setelah penghapusan
                break;
            case 5:
                editKaryaIlmiah(database);
                saveData(database, filename); // Simpan data setelah pengeditan
                break;
            case 6:
                sortirKaryaIlmiah(database);
                break;
            case 7:
                tampilkanStatistik(database);
                break;
            case 8:
                tampilkanStatistikTren(database);
                break;
            case 9:
                eksporDataCSV(database, "output.csv");
                break;
            case 10:
            {
                char confirm;
                std::cout << "\nAnda yakin ingin keluar? (y/n): ";
                std::cin >> confirm;
                if (confirm == 'y' || confirm == 'Y') {
                    running = false;
                    std::cout << "\nTerima kasih telah menggunakan program ini.\n";
                } else {
                    std::cout << "\nKembali ke menu utama.\n\n";
                }
                break;
            }
            default:
                std::cout << "\nPilihan menu tidak tersedia, silakan coba lagi.\n\n";
                break;
        }
    }

    return 0;
}
