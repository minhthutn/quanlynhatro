# quanlynhatro
phan mem quan ly nha tro 
#include <stdio.h>
#include <conio.h>
#include <string.h>
#include <stdlib.h>
#include <time.h>
#include <ctime>

#define MAX 100
#define LayTenChinh 10

typedef struct {
	char maKhach[10];
	char hoTen[50];
	char cccd[15];
	char sdt[15];
} NguoiThue;

// Thông tin Phòng Trọ
typedef struct {
	char maPhong[10];
	char tenPhong[30];
	int dienTich;   // mét vuông
	int giaPhong;
	char tienNghi[100];  // Tiện nghi (ví dụ "Điều hòa, Wifi")
	char trangThai[20]; // "Trong" / "Da thue"
	int soNguoiDangThue;
	char hoTen[50];
	char maKhach[10];
	char ngayThue[11];  // (dd/mm/yyyy)
	char ngayHetHanThue[11];  // (dd/mm/yyyy)
} PhongTro;

// Thông tin Hóa Đơn
typedef struct {
	char soHoaDon[10];
	char maKhach[10];
	char hoTen[50];
	char ngayLap[11]; //dd/mm/yyyy
	int tongTien;
} HoaDon;

// Thông tin Chi Tiết Hóa Đơn
typedef struct {
	char soHoaDon[10];
	char maPhong[10];
	int soNgayThue;
	int donGia;
} CTHoaDon;

typedef struct NodeNguoiThue {
	NguoiThue info;
	NodeNguoiThue* left, * right;
} NodeNguoiThue;

typedef struct NodePhongTro {
	PhongTro info;
	NodePhongTro* left, * right;
} NodePhongTro;

typedef struct NodeHoaDon {
	HoaDon info;
	NodeHoaDon* left, * right;
} NodeHoaDon;

typedef struct NodeCTHoaDon {
	CTHoaDon info;
	NodeCTHoaDon* left, * right;
} NodeCTHoaDon;

// ==== Prototype các hàm ====
void inPhongTroNhieuNguoiNhat(NodePhongTro* root, int maxNguoi);
void thongKeDoanhThuKhoangNgay(NodeHoaDon* root, int ngayBD, int thangBD, int namBD, int ngayKT, int thangKT, int namKT, int* tongDoanhThu);
NodeNguoiThue* timNguoiThue(NodeNguoiThue* root, char maKhach[]);
NodePhongTro* timPhongTro(NodePhongTro* root, char maPhong[]);
NodeHoaDon* timHoaDon(NodeHoaDon* root, char soHoaDon[]);

// KHỞI TẠO CÂY

NodeNguoiThue* createNodeNguoiThue(NguoiThue info) {
	NodeNguoiThue* temp = (NodeNguoiThue*)malloc(sizeof(NodeNguoiThue));
	temp->info = info;
	temp->left = temp->right = NULL;
	return temp;
}

NodePhongTro* createNodePhongTro(PhongTro info) {
	NodePhongTro* temp = (NodePhongTro*)malloc(sizeof(NodePhongTro));
	temp->info = info;
	temp->left = temp->right = NULL;
	return temp;
}

NodeHoaDon* createNodeHoaDon(HoaDon info) {
	NodeHoaDon* temp = (NodeHoaDon*)malloc(sizeof(NodeHoaDon));
	temp->info = info;
	temp->left = temp->right = NULL;
	return temp;
}

NodeCTHoaDon* createNodeCTHoaDon(CTHoaDon info) {
	NodeCTHoaDon* temp = (NodeCTHoaDon*)malloc(sizeof(NodeCTHoaDon));
	temp->info = info;
	temp->left = temp->right = NULL;
	return temp;
}

// HÀM THÊM

// Thêm Người thuê theo mã khách
NodeNguoiThue* insertNguoiThue(NodeNguoiThue* root, NguoiThue info) {
	if (root == NULL)
		return createNodeNguoiThue(info);
	if (strcmp(info.maKhach, root->info.maKhach) < 0)
		root->left = insertNguoiThue(root->left, info);
	else if (strcmp(info.maKhach, root->info.maKhach) > 0)
		root->right = insertNguoiThue(root->right, info);
	return root;
}

// Thêm Phòng trọ theo mã phòng
NodePhongTro* insertPhongTro(NodePhongTro* root, PhongTro info) {
	if (root == NULL)
		return createNodePhongTro(info);
	if (strcmp(info.maPhong, root->info.maPhong) < 0)
		root->left = insertPhongTro(root->left, info);
	else if (strcmp(info.maPhong, root->info.maPhong) > 0)
		root->right = insertPhongTro(root->right, info);
	return root;
}

// Thêm Hóa đơn theo số hóa đơn
NodeHoaDon* insertHoaDon(NodeHoaDon* root, HoaDon info) {
	if (root == NULL)
		return createNodeHoaDon(info);
	if (strcmp(info.soHoaDon, root->info.soHoaDon) < 0)
		root->left = insertHoaDon(root->left, info);
	else if (strcmp(info.soHoaDon, root->info.soHoaDon) > 0)
		root->right = insertHoaDon(root->right, info);
	return root;
}

// Thêm Chi tiết hóa đơn theo số hóa đơn + mã phòng
NodeCTHoaDon* insertCTHoaDon(NodeCTHoaDon* root, CTHoaDon info) {
	if (root == NULL)
		return createNodeCTHoaDon(info);
	int cmp = strcmp(info.soHoaDon, root->info.soHoaDon);
	if (cmp < 0 || (cmp == 0 && strcmp(info.maPhong, root->info.maPhong) < 0))
		root->left = insertCTHoaDon(root->left, info);
	else
		root->right = insertCTHoaDon(root->right, info);
	return root;
}

// Đọc người thuê từ file
void docFileNguoiThue(NodeNguoiThue** root) {
	FILE* f = fopen("nguoi_thue.txt", "r");
	if (f == NULL) {
		printf("\nKhong the mo file NguoiThue.txt\n");
		return;
	}
	NguoiThue nt;
	while (fscanf(f, "%9[^|]|%49[^|]|%14[^|]|%14[^\n]\n",
		nt.maKhach, nt.hoTen, nt.cccd, nt.sdt) == 4)
	{
		*root = insertNguoiThue(*root, nt);
	}
	fclose(f);
}

// Đọc phòng trọ từ file
void docFilePhongTro(NodePhongTro** root) {
	FILE* f = fopen("phong_tro.txt", "r");
	if (f == NULL) {
		printf("\nKhong the mo file phong_tro.txt\n");
		return;
	}
	PhongTro pt;
	while (fscanf(f, "%9[^|]|%29[^|]|%d|%d|%99[^|]|%19[^|]|%d|%49[^|]|%9[^|]|%10[^|]|%10[^\n]\n",
		pt.maPhong, pt.tenPhong, &pt.dienTich, &pt.giaPhong,
		pt.tienNghi, pt.trangThai, &pt.soNguoiDangThue,
		pt.hoTen, pt.maKhach, pt.ngayThue, pt.ngayHetHanThue) == 11)
	{
		*root = insertPhongTro(*root, pt);
	}
	fclose(f);
}

// Đọc hoá đơn từ file
void docFileHoaDon(NodeHoaDon** root) {
	FILE* f = fopen("hoa_don.txt", "r");
	if (f == NULL) {
		printf("\nKhong the mo file hoa_don.txt\n");
		return;
	}
	HoaDon hd;
	while (fscanf(f, "%[^|]|%[^|]|%[^|]|%[^|]|%d\n",
		hd.soHoaDon, hd.maKhach, hd.hoTen, hd.ngayLap, &hd.tongTien) == 5)
	{
		*root = insertHoaDon(*root, hd);
	}
	fclose(f);
}

// Đọc chi tiết hoá đơn từ file 
void docFileCTHoaDon(NodeCTHoaDon** root) {
	FILE* f = fopen("ct_hoa_don.txt", "r");
	if (f == NULL) {
		printf("\nKhong the mo file ct_hoa_don.txt\n");
		return;
	}
	CTHoaDon ct;
	while (fscanf(f, "%[^|]|%[^|]|%d|%d\n",
		ct.soHoaDon, ct.maPhong, &ct.soNgayThue, &ct.donGia) == 4)
	{
		*root = insertCTHoaDon(*root, ct);
	}
	fclose(f);
}

// Ghi người thuê vào file 
void ghiFileNguoiThue(NguoiThue nt) {
	FILE* f = fopen("nguoi_thue.txt", "a"); // a = append (ghi thêm)
	if (f == NULL) {
		printf("\nKhong the mo file nguoi_thue.txt de ghi.\n");
		return;
	}
	fprintf(f, "%s|%s|%s|%s\n", nt.maKhach, nt.hoTen, nt.cccd, nt.sdt);
	fclose(f);
}

// Ghi phòng trọ vào file
void ghiFilePhongTro(PhongTro pt) {
	FILE* f = fopen("phong_tro.txt", "a");
	if (f == NULL) {
		printf("\nKhong the mo file phong_tro.txt de ghi.\n");
		return;
	}
	fprintf(f, "%s|%s|%d|%d|%s|%s|%d|%s|%s|%s|%s\n",
		pt.maPhong, pt.tenPhong, pt.dienTich, pt.giaPhong, pt.tienNghi,
		pt.trangThai, pt.soNguoiDangThue, pt.hoTen, pt.maKhach,
		pt.ngayThue, pt.ngayHetHanThue);
	fclose(f);
}

// Ghi hoá đơn từ file
void ghiFileHoaDon(HoaDon hd) {
	FILE* f = fopen("hoa_don.txt", "a");
	if (f == NULL) {
		printf("\nKhong the mo file hoa_don.txt de ghi.\n");
		return;
	}
	fprintf(f, "%s|%s|%s|%s|%d\n",
		hd.soHoaDon, hd.maKhach, hd.hoTen, hd.ngayLap, hd.tongTien);
	fclose(f);
}

// Ghi chi tiết hoá đơn từ file 
void ghiFileCTHoaDon(CTHoaDon ct) {
	FILE* f = fopen("ct_hoa_don.txt", "a");
	if (f == NULL) {
		printf("\nKhong the mo file ct_hoa_don.txt de ghi.\n");
		return;
	}
	fprintf(f, "%s|%s|%d|%d\n", ct.soHoaDon, ct.maPhong, ct.soNgayThue, ct.donGia);
	fclose(f);
}

// Hiển thị nội dung file
void hienThiFilePhongTro()
{
	FILE* f = fopen("phong_tro.txt", "r");
	if (f == NULL) {
		printf("Khong the mo file phong_tro.txt\n");
		return;
	}
	char line[256];
	printf("\n--- Danh sach phong tro ---\n");
	while (fgets(line, sizeof(line), f)) {
		printf("%s", line);
	}
	fclose(f);
}

void hienThiFileNguoiThue()
{
	FILE* f = fopen("nguoi_thue.txt", "r");
	if (f == NULL) {
		printf("Khong the mo file nguoi_thue.txt\n");
		return;
	}
	char line[256];
	printf("\n--- Danh sach nguoi thue ---\n");
	while (fgets(line, sizeof(line), f)) {
		printf("%s", line);
	}
	fclose(f);
}

void hienThiFileHoaDon()
{
	FILE* f = fopen("hoa_don.txt", "r");
	if (f == NULL) {
		printf("Khong the mo file hoa_don.txt\n");
		return;
	}
	char line[256];
	printf("\n--- Danh sach hoa don ---\n");
	while (fgets(line, sizeof(line), f)) {
		printf("%s", line);
	}
	fclose(f);
}

// HÀM XÓA

// Xóa người thuê
NodeNguoiThue* xoaNguoiThue(NodeNguoiThue* root, char maKhach[]) {
	if (root == NULL) return NULL;
	int cmp = strcmp(maKhach, root->info.maKhach);
	if (cmp < 0) root->left = xoaNguoiThue(root->left, maKhach);
	else if (cmp > 0) root->right = xoaNguoiThue(root->right, maKhach);
	else {
		if (root->left == NULL) {
			NodeNguoiThue* temp = root->right;
			free(root);
			return temp;
		}
		else if (root->right == NULL) {
			NodeNguoiThue* temp = root->left;
			free(root);
			return temp;
		}
		NodeNguoiThue* temp = root->right;
		while (temp->left) temp = temp->left;
		root->info = temp->info;
		root->right = xoaNguoiThue(root->right, temp->info.maKhach);
	}
	return root;
}

void ghiLaiFileNguoiThue(NodeNguoiThue* root, FILE* f) {
	if (root == NULL) return;
	ghiLaiFileNguoiThue(root->left, f);
	fprintf(f, "%s|%s|%s|%s\n", root->info.maKhach, root->info.hoTen, root->info.cccd, root->info.sdt);
	ghiLaiFileNguoiThue(root->right, f);
}

void capNhatFileNguoiThue(NodeNguoiThue* root) {
	FILE* f = fopen("nguoi_thue.txt", "w");
	if (f == NULL) {
		printf("Khong the mo file nguoi_thue.txt de ghi!\n");
		return;
	}
	ghiLaiFileNguoiThue(root, f);
	fclose(f);
}

// Xóa phòng trọ
NodePhongTro* xoaPhongTro(NodePhongTro* root, char maPhong[]) {
	if (root == NULL) return NULL;
	int cmp = strcmp(maPhong, root->info.maPhong);
	if (cmp < 0) root->left = xoaPhongTro(root->left, maPhong);
	else if (cmp > 0) root->right = xoaPhongTro(root->right, maPhong);
	else {
		if (root->left == NULL) {
			NodePhongTro* temp = root->right;
			free(root);
			return temp;
		}
		else if (root->right == NULL) {
			NodePhongTro* temp = root->left;
			free(root);
			return temp;
		}
		NodePhongTro* temp = root->right;
		while (temp->left) temp = temp->left;
		root->info = temp->info;
		root->right = xoaPhongTro(root->right, temp->info.maPhong);
	}
	return root;
}

void ghiLaiFilePhongTro(NodePhongTro* root, FILE* f) {
	if (root == NULL) return;
	ghiLaiFilePhongTro(root->left, f);
	fprintf(f, "%s|%s|%d|%d|%s|%s|%d|%s|%s|%s|%s\n",
		root->info.maPhong, root->info.tenPhong, root->info.dienTich,
		root->info.giaPhong, root->info.tienNghi, root->info.trangThai,
		root->info.soNguoiDangThue, root->info.hoTen, root->info.maKhach,
		root->info.ngayThue, root->info.ngayHetHanThue);
	ghiLaiFilePhongTro(root->right, f);
}

void capNhatFilePhongTro(NodePhongTro* root) {
	FILE* f = fopen("phong_tro.txt", "w");
	if (f == NULL) {
		printf("Khong the mo file phong_tro.txt de ghi!\n");
		return;
	}
	ghiLaiFilePhongTro(root, f);
	fclose(f);
}

// Xóa hóa đơn
NodeHoaDon* xoaHoaDon(NodeHoaDon* root, char soHD[]) {
	if (root == NULL) return NULL;
	int cmp = strcmp(soHD, root->info.soHoaDon);
	if (cmp < 0) root->left = xoaHoaDon(root->left, soHD);
	else if (cmp > 0) root->right = xoaHoaDon(root->right, soHD);
	else {
		if (root->left == NULL) {
			NodeHoaDon* temp = root->right;
			free(root);
			return temp;
		}
		else if (root->right == NULL) {
			NodeHoaDon* temp = root->left;
			free(root);
			return temp;
		}
		NodeHoaDon* temp = root->right;
		while (temp->left) temp = temp->left;
		root->info = temp->info;
		root->right = xoaHoaDon(root->right, temp->info.soHoaDon);
	}
	return root;
}

void ghiLaiFileHoaDon(NodeHoaDon* root, FILE* f) {
	if (root == NULL) return;
	ghiLaiFileHoaDon(root->left, f);
	fprintf(f, "%s|%s|%s|%s|%d\n", root->info.soHoaDon, root->info.maKhach,
		root->info.hoTen, root->info.ngayLap, root->info.tongTien);
	ghiLaiFileHoaDon(root->right, f);
}

void capNhatFileHoaDon(NodeHoaDon* root) {
	FILE* f = fopen("hoa_don.txt", "w");
	if (f == NULL) {
		printf("Khong the mo file hoa_don.txt de ghi!\n");
		return;
	}
	ghiLaiFileHoaDon(root, f);
	fclose(f);
}

// HÀM THÊM
void themPhongTro(NodePhongTro** root) {
	PhongTro pt;
	printf("Nhap ma phong: ");
	scanf("%9s", pt.maPhong);
	getchar();
	if (timPhongTro(*root, pt.maPhong)) {
		printf("Ma phong da ton tai!\n");
		return;
	}

	printf("Nhap ten phong: ");
	fgets(pt.tenPhong, sizeof(pt.tenPhong), stdin);
	pt.tenPhong[strcspn(pt.tenPhong, "\n")] = 0;

	printf("Nhap dien tich (m2): ");
	scanf("%d", &pt.dienTich);

	printf("Nhap gia phong (VND/ngay): ");
	scanf("%d", &pt.giaPhong);
	getchar();

	printf("Nhap tien nghi (vi du: Dieu hoa, Wifi): ");
	fgets(pt.tienNghi, sizeof(pt.tienNghi), stdin);
	pt.tienNghi[strcspn(pt.tienNghi, "\n")] = 0;

	strcpy(pt.trangThai, "Trong");
	pt.soNguoiDangThue = 0;
	pt.hoTen[0] = '\0';
	pt.maKhach[0] = '\0';
	pt.ngayThue[0] = '\0';
	pt.ngayHetHanThue[0] = '\0';

	*root = insertPhongTro(*root, pt);
	ghiFilePhongTro(pt);

	printf("Them phong tro thanh cong!\n");
}


void themHoaDon(
	NodeHoaDon** rootHD, NodeCTHoaDon** rootCT,
	NodeNguoiThue** rootNT, NodePhongTro* rootPT)
{
	HoaDon hd;
	printf("\nNhap so hoa don: ");
	scanf("%s", hd.soHoaDon);

	if (timHoaDon(*rootHD, hd.soHoaDon)) {
		printf("So hoa don da ton tai!\n");
		return;
	}

	printf("Nhap ma khach: ");
	scanf("%s", hd.maKhach);

	NodeNguoiThue* foundKhach = timNguoiThue(*rootNT, hd.maKhach);
	NguoiThue nt;

	if (foundKhach == NULL) {
		printf("\nKhach hang moi. Moi nhap thong tin:\n");
		strcpy(nt.maKhach, hd.maKhach);

		printf("Nhap ho ten: ");
		getchar();
		fgets(nt.hoTen, sizeof(nt.hoTen), stdin);
		nt.hoTen[strcspn(nt.hoTen, "\n")] = 0;

		printf("Nhap CCCD: ");
		scanf("%s", nt.cccd);

		printf("Nhap so dien thoai: ");
		scanf("%s", nt.sdt);

		*rootNT = insertNguoiThue(*rootNT, nt);
		ghiFileNguoiThue(nt);
	}
	else {
		nt = foundKhach->info;
		printf("Khach hang da ton tai: %s\n", nt.hoTen);
	}
	strcpy(hd.hoTen, nt.hoTen);

	time_t now = time(0);
	tm* ltm = localtime(&now);
	sprintf(hd.ngayLap, "%02d/%02d/%04d", ltm->tm_mday, 1 + ltm->tm_mon, 1900 + ltm->tm_year);
	printf("Ngay lap hoa don: %s\n", hd.ngayLap);

	int soPhong, tongTien = 0;
	printf("Nhap so luong phong thue: ");
	scanf("%d", &soPhong);

	for (int i = 0; i < soPhong; i++) {
		CTHoaDon ct;
		strcpy(ct.soHoaDon, hd.soHoaDon);

		printf("\nNhap ma phong %d: ", i + 1);
		scanf("%s", ct.maPhong);

		NodePhongTro* pPhong = timPhongTro(rootPT, ct.maPhong);
		if (pPhong == NULL) {
			printf("Phong khong ton tai!\n");
			i--;
			continue;
		}
		if (strcmp(pPhong->info.trangThai, "DaThue") == 0) {
			printf("Phong da duoc thue!\n");
			i--;
			continue;
		}

		printf("Nhap so ngay thue: ");
		scanf("%d", &ct.soNgayThue);
		if (ct.soNgayThue <= 0) {
			printf("So ngay thue khong hop le!\n");
			i--;
			continue;
		}

		ct.donGia = pPhong->info.giaPhong;
		tongTien += ct.donGia * ct.soNgayThue;

		*rootCT = insertCTHoaDon(*rootCT, ct);
		ghiFileCTHoaDon(ct);

		// Cập nhật trạng thái phòng
		strcpy(pPhong->info.trangThai, "DaThue");
		pPhong->info.soNguoiDangThue = 1;
		strcpy(pPhong->info.maKhach, nt.maKhach);
		strcpy(pPhong->info.hoTen, nt.hoTen);
		strcpy(pPhong->info.ngayThue, hd.ngayLap);
		strcpy(pPhong->info.ngayHetHanThue, "");
	}

	hd.tongTien = tongTien;
	*rootHD = insertHoaDon(*rootHD, hd);
	ghiFileHoaDon(hd);

	capNhatFilePhongTro(rootPT);

	printf("\nThem hoa don thanh cong!\n");
}

// HÀM TÌM KIẾM
NodeNguoiThue* timNguoiThue(NodeNguoiThue* root, char maKhach[]) {
	if (root == NULL) return NULL;
	int cmp = strcmp(maKhach, root->info.maKhach);
	if (cmp == 0)
		return root;
	else if (cmp < 0)
		return timNguoiThue(root->left, maKhach);
	else
		return timNguoiThue(root->right, maKhach);
}

NodePhongTro* timPhongTro(NodePhongTro* root, char maPhong[]) {
	if (root == NULL) return NULL;
	int cmp = strcmp(maPhong, root->info.maPhong);
	if (cmp == 0)
		return root;
	else if (cmp < 0)
		return timPhongTro(root->left, maPhong);
	else
		return timPhongTro(root->right, maPhong);
}

NodeHoaDon* timHoaDon(NodeHoaDon* root, char soHoaDon[]) {
	if (root == NULL) return NULL;

	int cmp = strcmp(soHoaDon, root->info.soHoaDon);
	if (cmp == 0)
		return root;
	else if (cmp < 0)
		return timHoaDon(root->left, soHoaDon);
	else
		return timHoaDon(root->right, soHoaDon);
}

void timSoNguoiThueNhieuNhat(NodePhongTro* root, int* maxNguoi) {
	if (root == NULL) return;
	if (root->info.soNguoiDangThue > *maxNguoi) {
		*maxNguoi = root->info.soNguoiDangThue;
	}
	timSoNguoiThueNhieuNhat(root->left, maxNguoi);
	timSoNguoiThueNhieuNhat(root->right, maxNguoi);
}

void timPhongNhieuNguoiThueNhat(NodePhongTro* root) {
	if (root == NULL) {
		printf("\nDanh sach phong tro rong!\n");
		return;
	}
	int maxNguoi = 0;
	timSoNguoiThueNhieuNhat(root, &maxNguoi);
	inPhongTroNhieuNguoiNhat(root, maxNguoi);
	printf("\nSo nguoi thue nhieu nhat: %d\n", maxNguoi);
}

void inPhongTroNhieuNguoiNhat(NodePhongTro* root, int maxNguoi) {
	if (root == NULL) return;
	inPhongTroNhieuNguoiNhat(root->left, maxNguoi);
	if (root->info.soNguoiDangThue == maxNguoi) {
		printf("\n--- Phong co nhieu nguoi thue ---\n");
		printf("Ma Phong: %s\n", root->info.maPhong);
		printf("So Nguoi Dang Thue: %d\n", root->info.soNguoiDangThue);
		printf("Dien Tich: %d\n", root->info.dienTich);
		printf("Gia Phong: %d\n", root->info.giaPhong);
	}

	inPhongTroNhieuNguoiNhat(root->right, maxNguoi);
}

// HÀM SẮP XẾP

// Sắp xếp hóa đơn giảm dần theo tổng tiền
void duaHoaDonVaoMang(NodeHoaDon* root, HoaDon arr[], int& n) {
	if (root == NULL) return;
	duaHoaDonVaoMang(root->left, arr, n);
	arr[n++] = root->info;
	duaHoaDonVaoMang(root->right, arr, n);
}

void sapXepHoaDonGiamTongTien(HoaDon arr[], int n) {
	for (int i = 0; i < n - 1; i++) {
		for (int j = i + 1; j < n; j++) {
			if (arr[i].tongTien < arr[j].tongTien) {
				HoaDon temp = arr[i];
				arr[i] = arr[j];
				arr[j] = temp;
			}
		}
	}
}

void inDanhSachHoaDon(HoaDon arr[], int n) {
	printf("%-10s %-10s %-25s %-15s %-10s\n", "SoHoaDon", "MaKhach", "HoTen", "NgayLap", "TongTien");
	for (int i = 0; i < n; i++) {
		printf("%-10s %-10s %-25s %-15s %-10d\n",
			arr[i].soHoaDon,
			arr[i].maKhach,
			arr[i].hoTen,
			arr[i].ngayLap,
			arr[i].tongTien);
	}
}

// Sắp xếp hóa đơn tăng dần theo tên, nếu trùng thì dùng mã khách thuê 
void layTenChinh(char hoTen[], char tenChinh[]) {
	if (hoTen == NULL) {
		strcpy(tenChinh, "");
		return;
	}

	int len = strlen(hoTen);
	if (len == 0) {
		strcpy(tenChinh, "");
		return;
	}

	int i = len - 1;
	while (i >= 0 && hoTen[i] != ' ') {
		i--;
	}
	strcpy(tenChinh, hoTen + i + 1);
}

NodeHoaDon* insertTheoTen(NodeHoaDon* root, HoaDon hd) {
	if (root == NULL) {
		NodeHoaDon* p = (NodeHoaDon*)malloc(sizeof(NodeHoaDon));
		p->info = hd;
		p->left = p->right = NULL;
		return p;
	}
	char tenMoi[50], tenRoot[50];
	layTenChinh(hd.hoTen, tenMoi);
	layTenChinh(root->info.hoTen, tenRoot);

	int cmp = strcmp(tenMoi, tenRoot);
	if (cmp < 0) {
		root->left = insertTheoTen(root->left, hd);
	}
	else if (cmp > 0) {
		root->right = insertTheoTen(root->right, hd);
	}
	else {
		// Nếu tên chính trùng nhau thì so tiếp theo maKhach
		if (strcmp(hd.maKhach, root->info.maKhach) < 0)
			root->left = insertTheoTen(root->left, hd);
		else
			root->right = insertTheoTen(root->right, hd);
	}
	return root;
}

void chenCayTheoTen(NodeHoaDon* rootHD, NodeHoaDon** rootTheoTen) {
	if (rootHD == NULL) return;
	chenCayTheoTen(rootHD->left, rootTheoTen);
	*rootTheoTen = insertTheoTen(*rootTheoTen, rootHD->info);
	chenCayTheoTen(rootHD->right, rootTheoTen);
}

void inCayHoaDon(NodeHoaDon* root) {
	if (root == NULL) return;
	inCayHoaDon(root->left);
	printf("%-10s %-10s %-25s %-15s %-10d\n",
		root->info.soHoaDon,
		root->info.maKhach,
		root->info.hoTen,
		root->info.ngayLap,
		root->info.tongTien);
	inCayHoaDon(root->right);
}

// HÀM TÌM HOÁ ĐƠN VÀ HIÊN THỊ THEO SỐ HOÁ ĐƠN
void inHoaDon(HoaDon hd) {
	printf("\n--- Hoa Don ---\n");
	printf("So Hoa Don: %s\n", hd.soHoaDon);
	printf("Ma Khach: %s\n", hd.maKhach);
	printf("Ho ten: %s\n", hd.hoTen);
	printf("Ngay Lap: %s\n", hd.ngayLap);
	printf("Tong Tien: %d\n", hd.tongTien);
}

void inCTHoaDonTheoSoHD(NodeCTHoaDon* root, char soHoaDon[]) {
	if (root == NULL) return;
	inCTHoaDonTheoSoHD(root->left, soHoaDon);
	if (strcmp(root->info.soHoaDon, soHoaDon) == 0) {
		printf("\nChi Tiet Hoa Don:\n");
		printf("Ma Phong: %s\n", root->info.maPhong);
		printf("So Ngay Thue: %d\n", root->info.soNgayThue);
		printf("Don Gia: %d\n", root->info.donGia);
	}
	inCTHoaDonTheoSoHD(root->right, soHoaDon);
}

void timKiemHoaDonTheoSoHD(NodeHoaDon* rootHD, NodeCTHoaDon* rootCT) {
	char soHD[20];
	printf("\nNhap so hoa don can tim: ");
	scanf("%s", soHD);

	NodeHoaDon* p = timHoaDon(rootHD, soHD);
	if (p != NULL) {
		inHoaDon(p->info);
		inCTHoaDonTheoSoHD(rootCT, soHD);
	}
	else
		printf("\nKhong tim thay hoa don!\n");
}

// HÀM LIÊT KÊ HOÁ ĐƠN THEO MÃ KHÁCH HÀNG
void lietKeHoaDonTheoMaKhach(NodeHoaDon* rootHD, NodeCTHoaDon* rootCT, char maKhach[],int*timThay)
{
	int timThay = 0;
	if (rootHD == NULL) return;
	lietKeHoaDonTheoMaKhach(rootHD->left, rootCT, maKhach);
	if (strcmp(rootHD->info.maKhach, maKhach) == 0) {
		inHoaDon(rootHD->info);
		inCTHoaDonTheoSoHD(rootCT, rootHD->info.soHoaDon);
		*timThay = 1;
	}
	lietKeHoaDonTheoMaKhach(rootHD->right, rootCT, maKhach);
}

void lietKeHoaDonTheoMaKH(NodeHoaDon* rootHD, NodeCTHoaDon* rootCT) {
	char maKhach[20];
	printf("\nNhap ma khach hang can liet ke hoa don: ");
	scanf("%s", maKhach);

	int timThay = 0;
	printf("\n--- Danh sach hoa don cua khach %s ---\n", maKhach);
	lietKeHoaDonTheoMaKhach(rootHD, rootCT, maKhach);

	if (!timThay)
		printf("Khach hang %s khong co hoa don nao.\n", maKhach);
}

// HÀM THỐNG KÊ DOANH THU
void tachNgayThangNam(char ngayLap[], int& ngay, int& thang, int& nam) {
	sscanf(ngayLap, "%d/%d/%d", &ngay, &thang, &nam);
}

int soSanhNgay(int ngay1, int thang1, int nam1, int ngay2, int thang2, int nam2) {
	if (nam1 != nam2) return nam1 - nam2;
	if (thang1 != thang2) return thang1 - thang2;
	return ngay1 - ngay2;
}

int doanhThuTheoNgay(NodeHoaDon* root, int ngay, int thang, int nam) {
	if (root == NULL) return 0;
	int d, m, y;
	tachNgayThangNam(root->info.ngayLap, d, m, y);
	int tong = 0;
	if (d == ngay && m == thang && y == nam) {
		tong += root->info.tongTien;
	}
	tong += doanhThuTheoNgay(root->left, ngay, thang, nam);
	tong += doanhThuTheoNgay(root->right, ngay, thang, nam);
	return tong;
}

int doanhThuTheoThang(NodeHoaDon* root, int thang, int nam) {
	if (root == NULL) return 0;
	int d, m, y;
	tachNgayThangNam(root->info.ngayLap, d, m, y);
	int tong = 0;
	if (m == thang && y == nam)
		tong += root->info.tongTien;

	tong += doanhThuTheoThang(root->left, thang, nam);
	tong += doanhThuTheoThang(root->right, thang, nam);
	return tong;
}

int doanhThuTheoQuy(NodeHoaDon* root, int quy, int nam) {
	if (root == NULL) return 0;
	int d, m, y;
	tachNgayThangNam(root->info.ngayLap, d, m, y);
	int tong = 0;
	if (y == nam) {
		if ((quy == 1 && (m == 1 || m == 2 || m == 3)) ||
			(quy == 2 && (m == 4 || m == 5 || m == 6)) ||
			(quy == 3 && (m == 7 || m == 8 || m == 9)) ||
			(quy == 4 && (m == 10 || m == 11 || m == 12))) {
			tong += root->info.tongTien;
		}
	}
	tong += doanhThuTheoQuy(root->left, quy, nam);
	tong += doanhThuTheoQuy(root->right, quy, nam);
	return tong;
}

int doanhThuTheoNam(NodeHoaDon* root, int nam) {
	if (root == NULL) return 0;
	int d, m, y;
	tachNgayThangNam(root->info.ngayLap, d, m, y);
	int tong = 0;
	if (y == nam)
		tong += root->info.tongTien;

	tong += doanhThuTheoNam(root->left, nam);
	tong += doanhThuTheoNam(root->right, nam);
	return tong;
}

int doanhThuTuNgayDenNgay(NodeHoaDon* root, int ngayBD, int thangBD, int namBD, int ngayKT, int thangKT, int namKT) {
	if (root == NULL) return 0;
	int d, m, y;
	tachNgayThangNam(root->info.ngayLap, d, m, y);
	int tong = 0;
	if (soSanhNgay(d, m, y, ngayBD, thangBD, namBD) >= 0 && soSanhNgay(d, m, y, ngayKT, thangKT, namKT) <= 0)
		tong += root->info.tongTien;

	tong += doanhThuTuNgayDenNgay(root->left, ngayBD, thangBD, namBD, ngayKT, thangKT, namKT);
	tong += doanhThuTuNgayDenNgay(root->right, ngayBD, thangBD, namBD, ngayKT, thangKT, namKT);
	return tong;
}

void thongKeDoanhThu(NodeHoaDon* root) {
	int choice;
	do
	{
		printf("\nThong ke doanh thu theo:\n");
		printf("1. Ngay\n");
		printf("2. Thang\n");
		printf("3. Quy\n");
		printf("4. Nam\n");
		printf("5. Tu ngay den ngay\n");
		printf("0. Thoat\n");
		printf("Chon: ");
		scanf("%d", &choice);

		int ngayBD, thangBD, namBD, ngayKT, thangKT, namKT;
		int tongDoanhThu = 0;

		if (choice == 1) {
			int d, m, y;
			printf("Nhap ngay/thang/nam: ");
			scanf("%d/%d/%d", &d, &m, &y);
			printf("Tong doanh thu: %d\n", doanhThuTheoNgay(root, d, m, y));
		}
		else if (choice == 2) {
			int m, y;
			printf("Nhap thang/nam: ");
			scanf("%d/%d", &m, &y);
			printf("Tong doanh thu: %d\n", doanhThuTheoThang(root, m, y));
		}
		else if (choice == 3) {
			int q, y;
			do {
				printf("Nhap lua chon quy (1-4) va nam: ");
				scanf("%d %d", &q, &y);
				if (q < 1 || q > 4) {
					printf("Quy khong hop le. Vui long nhap lai!\n");
				}
			} while (q < 1 || q > 4);
			printf("Tong doanh thu: %d\n", doanhThuTheoQuy(root, q, y));
		}
		else if (choice == 4) {
			int y;
			printf("Nhap nam: ");
			scanf("%d", &y);
			printf("Tong doanh thu: %d\n", doanhThuTheoNam(root, y));
		}
		else if (choice == 5) {
			int d1, m1, y1, d2, m2, y2;
			printf("Nhap ngay bat dau (d/m/y): ");
			scanf("%d/%d/%d", &d1, &m1, &y1);
			printf("Nhap ngay ket thuc (d/m/y): ");
			scanf("%d/%d/%d", &d2, &m2, &y2);
			printf("Tong doanh thu: %d\n", doanhThuTuNgayDenNgay(root, d1, m1, y1, d2, m2, y2));
		}
	} while (choice != 0);
}

// HÀM LỌC
int kiemTraGiaPhong(int giaPhong, int giaMin, int giaMax) {
	return giaPhong >= giaMin && giaPhong <= giaMax;
}

int kiemTraDienTich(int dienTich, int dienTichMin, int dienTichMax) {
	return dienTich >= dienTichMin && dienTich <= dienTichMax;
}

int kiemTraTienNghi(char* tienNghi, const char* tienNghiCanTim) {
	return strstr(tienNghi, tienNghiCanTim) != NULL;
}

void locPhongTro(NodePhongTro* root, int giaMin, int giaMax, int dienTichMin, int dienTichMax, const char* tienNghiCanTim, int* coPhong) {
	if (root == NULL) return;

	if (kiemTraGiaPhong(root->info.giaPhong, giaMin, giaMax) &&
		kiemTraDienTich(root->info.dienTich, dienTichMin, dienTichMax) &&
		kiemTraTienNghi(root->info.tienNghi, tienNghiCanTim)) {

		printf("%-10s %-20s %-10d %-8d %-20s\n",
			root->info.maPhong,
			root->info.hoTen,
			root->info.giaPhong,
			root->info.dienTich,
			root->info.tienNghi);
		(*coPhong)++;
	}
	locPhongTro(root->left, giaMin, giaMax, dienTichMin, dienTichMax, tienNghiCanTim, coPhong);
	locPhongTro(root->right, giaMin, giaMax, dienTichMin, dienTichMax, tienNghiCanTim, coPhong);
}

// CẢNH BÁO PHÒNG
void canhBaoPhongTrong(NodePhongTro* root, int* coPhongTrong) {
	if (root == NULL) return;
	canhBaoPhongTrong(root->left, coPhongTrong);
	if (strcmp(root->info.trangThai, "Trong") == 0) {
		printf("%-10s %-20s %-8d %-10d %-20s\n",
			root->info.maPhong,
			root->info.tenPhong,
			root->info.dienTich,
			root->info.giaPhong,
			root->info.tienNghi);
		*coPhongTrong = 1;
	}
	canhBaoPhongTrong(root->right, coPhongTrong);
}

// Hàm cảnh báo phòng đã hết hạn
int phongDaHetHan(PhongTro p, int ngayHT, int thangHT, int namHT) {
	if (strcmp(p.trangThai, "DaThue") != 0) return 0;
	int d, m, y;
	tachNgayThangNam(p.ngayHetHanThue, d, m, y);
	return soSanhNgay(ngayHT, thangHT, namHT, d, m, y) > 0;
}

void canhBaoPhongHetHan(NodePhongTro* root, int ngayHT, int thangHT, int namHT, int* coPhongHetHan) {
	if (root == NULL) return;
	canhBaoPhongHetHan(root->left, ngayHT, thangHT, namHT, coPhongHetHan);
	if (phongDaHetHan(root->info, ngayHT, thangHT, namHT)) {
		printf("%-10s %-20s %-20s %-12s\n",
			root->info.maPhong,
			root->info.tenPhong,
			root->info.hoTen,
			root->info.ngayHetHanThue);
		*coPhongHetHan = 1;
	}
	canhBaoPhongHetHan(root->right, ngayHT, thangHT, namHT, coPhongHetHan);
}

void menu() {
	printf("\n=================== MENU QUAN LY THUE PHONG ===================\n");
	printf("1. Them thong tin mot phong tro moi\n");
	printf("2. Them hoa don moi va cap nhat chi tiet hoa don, nguoi thue moi\n");
	printf("3. Hien thi danh sach phong tro\n");
	printf("4. Hien thi danh sach nguoi thue\n");
	printf("5. Hien thi danh sach hoa don\n");
	printf("6. Sap xep hoa don giam dan theo tong tien\n");
	printf("7. Sap xep hoa don tang dan theo ten nguoi thue\n");
	printf("8. Tim kiem va hien thi thong tin hoa don theo so hoa don\n");
	printf("9. Tim phong tro co so nguoi thue nhieu nhat\n");
	printf("10. Liet ke danh sach hoa don chi tiet theo ma khach hang\n");
	printf("11. Loc phong tro theo gia, dien tich, tien nghi\n");
	printf("12. Thong ke doanh thu cho thue\n");
	printf("13. Canh bao phong het han thue va phong trong\n");
	printf("14. Xoa nguoi thue theo ma\n");
	printf("15. Xoa phong tro theo ma\n");
	printf("16. Xoa hoa don theo so hoa don\n");
	printf("0. Thoat chuong trinh\n");
	printf("===============================================================\n");
}

NodeNguoiThue* rootNT = NULL;
NodePhongTro* rootPT = NULL;
NodeHoaDon* rootHD = NULL;
NodeCTHoaDon* rootCT = NULL;
NodeHoaDon* rootTheoTen = NULL;

void process()
{
	int chon;
	HoaDon arr[MAX];
	int soPhong = 0;
	int coPhong = 0;
	int n = 0;
	do
	{
		menu();
		printf("Nhap lua chon: ");
		scanf("%d", &chon);

		switch (chon)
		{
		case 1:
			themPhongTro(&rootPT);
			break;
		case 2:
			themHoaDon(&rootHD, &rootCT, &rootNT, rootPT);
			break;
		case 3:
			hienThiFilePhongTro();
			break;
		case 4:
			hienThiFileNguoiThue();
			break;
		case 5:
			hienThiFileHoaDon();
			break;
		case 6:
			duaHoaDonVaoMang(rootHD, arr, n);

			if (n == 0) {
				printf("Danh sach hoa don rong!\n");
			}
			else {
				sapXepHoaDonGiamTongTien(arr, n);
				inDanhSachHoaDon(arr, n);
			}
			break;
		case 7:
			rootTheoTen = NULL;
			chenCayTheoTen(rootHD, &rootTheoTen);
			printf("%-10s %-10s %-25s %-15s %-10s\n", "SoHoaDon", "MaKhach", "HoTen", "NgayLap", "TongTien");
			printf("-------------------------------------------------------------------------------\n");
			inCayHoaDon(rootTheoTen);
			break;
		case 8:
			timKiemHoaDonTheoSoHD(rootHD, rootCT);
			break;
		case 9:
			timPhongNhieuNguoiThueNhat(rootPT);
			break;
		case 10:
			lietKeHoaDonTheoMaKH(rootHD, rootCT);
			break;
		case 11:
			int giaMin, giaMax, dienTichMin, dienTichMax;
			char tienNghiCanTim[100];

			printf("Nhap gia phong toi thieu: ");
			scanf("%d", &giaMin);
			printf("Nhap gia phong toi da: ");
			scanf("%d", &giaMax);
			printf("Nhap dien tich toi thieu: ");
			scanf("%d", &dienTichMin);
			printf("Nhap dien tich toi da: ");
			scanf("%d", &dienTichMax);
			printf("Nhap tien nghi can tim: ");
			scanf("%s", tienNghiCanTim);

			printf("\n===== DANH SACH PHONG TRO LOC DUOC =====\n");
			printf("%-10s %-20s %-10s %-8s %-20s\n", "MaPhong", "TenKhach", "GiaPhong", "DienTich", "TienNghi");

			locPhongTro(rootPT, giaMin, giaMax, dienTichMin, dienTichMax, tienNghiCanTim, &coPhong);

			if (coPhong == 0)
				printf("Khong tim thay phong tro phu hop voi tieu chi loc.\n");
			break;
		case 12:
			if (rootHD != NULL)
				thongKeDoanhThu(rootHD);
			else
				printf("Khong co du lieu hoa don de thong ke!\n");
			break;
		case 13: {
			time_t rawtime;
			struct tm* timeinfo;

			// Lấy thời gian hiện tại theo giờ UTC
			time(&rawtime);

			// Cộng thêm 7 giờ để được GMT+7
			rawtime += 7 * 3600;

			// Chuyển sang struct tm theo localtime GMT+7
			timeinfo = gmtime(&rawtime);

			int ngayHT = timeinfo->tm_mday;
			int thangHT = timeinfo->tm_mon + 1;  // tháng tính từ 0
			int namHT = timeinfo->tm_year + 1900; // năm tính từ 1900

			printf("Ngay hien tai: %02d/%02d/%d\n", ngayHT, thangHT, namHT);

			printf("\n===== CAC PHONG CON TRONG =====\n");
			printf("%-10s %-20s %-8s %-10s %-20s\n",
				"MaPhong", "TenPhong", "DienTich", "GiaTien", "TienNghi");

			int coPhongTrong = 0;
			canhBaoPhongTrong(rootPT, &coPhongTrong);
			if (!coPhongTrong) {
				printf("Khong co phong trong.\n");
			}

			printf("\n===== CAC PHONG DA HET HAN =====\n");
			printf("%-10s %-20s %-20s %-12s\n",
				"MaPhong", "TenPhong", "KhachHang", "HetHan");

			int coPhongHetHan = 0;
			canhBaoPhongHetHan(rootPT, ngayHT, thangHT, namHT, &coPhongHetHan);
			if (!coPhongHetHan) {
				printf("Khong co phong nao het han.\n");
			}

			break;
		}

		case 14:
			char maKhach[10];
			printf("Nhap ma khach can xoa: ");
			scanf("%s", maKhach);
			rootNT = xoaNguoiThue(rootNT, maKhach);
			capNhatFileNguoiThue(rootNT);
			printf("=> Da xoa nguoi thue %s\n", maKhach);
			break;
		case 15:
			char maPhong[10];
			printf("Nhap ma phong can xoa: ");
			scanf("%s", maPhong);
			rootPT = xoaPhongTro(rootPT, maPhong);
			capNhatFilePhongTro(rootPT);
			printf("=> Da xoa phong %s\n", maPhong);
			break;
		case 16:
			char soHD[10];
			printf("Nhap so hoa don can xoa: ");
			scanf("%s", soHD);
			rootHD = xoaHoaDon(rootHD, soHD);
			capNhatFileHoaDon(rootHD);
			printf("=> Da xoa hoa don %s\n", soHD);
			break;
		case 0:
			printf("\nThoat chuong trinh...\n");
			break;
		default:
			printf("\nLua chon khong hop le. Vui long chon lai!\n");
			break;
		}
	} while (chon != 0);
}

void main()
{
	docFileNguoiThue(&rootNT);
	docFilePhongTro(&rootPT);
	docFileHoaDon(&rootHD);
	docFileCTHoaDon(&rootCT);

	process();

	getch();;
}
