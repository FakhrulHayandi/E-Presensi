# Tambahkan fitur rekap bulanan ke dalam file HTML sebelumnya

html_with_rekap = """<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>PRESENSI PPPK</title>
<style>
    body { font-family: Arial, sans-serif; margin: 20px; background: #f7f7f7; }
    h1 { color: #333; }
    label { display: block; margin-top: 10px; font-weight: bold; }
    input, select, button { padding: 8px; margin-top: 5px; width: 100%; max-width: 400px; }
    table { border-collapse: collapse; width: 100%; margin-top: 20px; background: #fff; }
    table, th, td { border: 1px solid #ccc; }
    th, td { padding: 8px; text-align: center; }
    th { background: #eee; }
    .btn { background: #4CAF50; color: white; border: none; cursor: pointer; margin-top: 10px; }
    .btn:hover { background: #45a049; }
    .container { max-width: 900px; margin: auto; }
</style>
</head>
<body onload="document.getElementById('nama').focus();">

<div class="container">
    <h1>PRESENSI PPPK</h1>
    <p>Data tersimpan di browser (localStorage). Unduh file Excel untuk backup.</p>

    <label>Nama Pegawai</label>
    <input type="text" id="nama" placeholder="Masukkan nama" onkeydown="if(event.key==='Enter'){document.getElementById('nip').focus();}">

    <label>NIP</label>
    <input type="text" id="nip" placeholder="Masukkan NIP" onkeydown="if(event.key==='Enter'){absenMasuk();}">

    <label>Status Kehadiran</label>
    <select id="status">
        <option value="Hadir">Hadir</option>
        <option value="Izin">Izin</option>
        <option value="Sakit">Sakit</option>
        <option value="Alpha">Alpha</option>
    </select>

    <button class="btn" onclick="absenMasuk()">Absen Masuk</button>
    <button class="btn" onclick="absenPulang()">Absen Pulang</button>
    <button class="btn" onclick="resetData()">Reset Semua Data</button>
    <button class="btn" onclick="unduhExcel()">Unduh Excel</button>
    <button class="btn" onclick="tampilkanRekapBulanan()">Lihat Rekap Bulanan</button>

    <h2>Absensi Harian</h2>
    <table id="tabelPresensi">
        <thead>
            <tr>
                <th>No</th>
                <th>Nama</th>
                <th>NIP</th>
                <th>Tanggal</th>
                <th>Jam Masuk</th>
                <th>Jam Pulang</th>
                <th>Keterangan</th>
                <th>Status</th>
            </tr>
        </thead>
        <tbody></tbody>
    </table>

    <h2>Rekap Bulanan</h2>
    <table id="tabelRekap" style="display:none;">
        <thead>
            <tr>
                <th>No</th>
                <th>Nama</th>
                <th>NIP</th>
                <th>Bulan</th>
                <th>Hadir</th>
                <th>Izin</th>
                <th>Sakit</th>
                <th>Alpha</th>
            </tr>
        </thead>
        <tbody></tbody>
    </table>
</div>

<script>
    let presensi = JSON.parse(localStorage.getItem("presensiPPPK")) || [];

    function simpanData() {
        localStorage.setItem("presensiPPPK", JSON.stringify(presensi));
        tampilkanData();
    }

    function tampilkanData() {
        let tbody = document.querySelector("#tabelPresensi tbody");
        tbody.innerHTML = "";
        presensi.forEach((item, index) => {
            let tr = `<tr>
                <td>${index + 1}</td>
                <td>${item.nama}</td>
                <td>${item.nip}</td>
                <td>${item.tanggal}</td>
                <td>${item.jamMasuk || ""}</td>
                <td>${item.jamPulang || ""}</td>
                <td>${item.keterangan || ""}</td>
                <td>${item.status}</td>
            </tr>`;
            tbody.innerHTML += tr;
        });
    }

    function absenMasuk() {
        let nama = document.getElementById("nama").value.trim();
        let nip = document.getElementById("nip").value.trim();
        let status = document.getElementById("status").value;
        if (!nama || !nip) {
            alert("Nama dan NIP wajib diisi!");
            return;
        }
        let tanggal = new Date().toLocaleDateString("id-ID");
        let jam = new Date().toLocaleTimeString("id-ID");
        presensi.push({
            nama, nip, tanggal,
            jamMasuk: jam,
            jamPulang: "",
            keterangan: "",
            status
        });
        simpanData();
        document.getElementById("nama").value = "";
        document.getElementById("nip").value = "";
        document.getElementById("nama").focus();
    }

    function absenPulang() {
        let nama = document.getElementById("nama").value.trim();
        let nip = document.getElementById("nip").value.trim();
        let tanggal = new Date().toLocaleDateString("id-ID");
        let jam = new Date().toLocaleTimeString("id-ID");

        let data = presensi.find(item => item.nama === nama && item.nip === nip && item.tanggal === tanggal);
        if (data) {
            data.jamPulang = jam;
            simpanData();
        } else {
            alert("Data absen masuk belum ada untuk hari ini!");
        }
        document.getElementById("nama").value = "";
        document.getElementById("nip").value = "";
        document.getElementById("nama").focus();
    }

    function resetData() {
        if (confirm("Yakin hapus semua data?")) {
            presensi = [];
            simpanData();
        }
    }

    function unduhExcel() {
        let csv = "No,Nama,NIP,Tanggal,Jam Masuk,Jam Pulang,Keterangan,Status\\n";
        presensi.forEach((item, index) => {
            csv += `${index + 1},${item.nama},${item.nip},${item.tanggal},${item.jamMasuk || ""},${item.jamPulang || ""},${item.keterangan || ""},${item.status}\\n`;
        });
        let blob = new Blob([csv], { type: "text/csv;charset=utf-8;" });
        let link = document.createElement("a");
        link.href = URL.createObjectURL(blob);
        link.download = "Presensi_PPPK.csv";
        link.click();
    }

    function tampilkanRekapBulanan() {
        let tbody = document.querySelector("#tabelRekap tbody");
        tbody.innerHTML = "";
        document.getElementById("tabelRekap").style.display = "table";

        let rekap = {};

        presensi.forEach(item => {
            let bulan = item.tanggal.split("/")[1] + "-" + item.tanggal.split("/")[2]; // format MM-YYYY
            let key = item.nama + "|" + item.nip + "|" + bulan;
            if (!rekap[key]) {
                rekap[key] = { nama: item.nama, nip: item.nip, bulan: bulan, Hadir: 0, Izin: 0, Sakit: 0, Alpha: 0 };
            }
            rekap[key][item.status]++;
        });

        let index = 1;
        for (let key in rekap) {
            let r = rekap[key];
            let tr = `<tr>
                <td>${index++}</td>
                <td>${r.nama}</td>
                <td>${r.nip}</td>
                <td>${r.bulan}</td>
                <td>${r.Hadir}</td>
                <td>${r.Izin}</td>
                <td>${r.Sakit}</td>
                <td>${r.Alpha}</td>
            </tr>`;
            tbody.innerHTML += tr;
        }
    }

    tampilkanData();
</script>

</body>
</html>
"""

# Simpan sebagai ZIP siap upload
import zipfile, os

os.makedirs("/mnt/data/presensi-pppk-rekap", exist_ok=True)
with open("/mnt/data/presensi-pppk-rekap/index.html", "w", encoding="utf-8") as f:
    f.write(html_with_rekap)

zip_path_rekap = "/mnt/data/presensi-pppk-rekap.zip"
with zipfile.ZipFile(zip_path_rekap, 'w') as zipf:
    zipf.write("/mnt/data/presensi-pppk-rekap/index.html", "index.html")

zip_path_rekap
