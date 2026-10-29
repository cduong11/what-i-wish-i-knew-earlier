# Hướng dẫn Command Line cho người mới bắt đầu

Oct 29, 2026 · @Chau Duong

## 1. Giới thiệu

Command line (dòng lệnh) là cách làm việc với máy tính bằng cách gõ lệnh thay vì bấm chuột. Trong phân tích dữ liệu, nhất là dữ liệu sinh học quy mô lớn, đây gần như là kỹ năng nền tảng: phần lớn công cụ (STAR, samtools, Nextflow...) chỉ chạy qua dòng lệnh, và server hay cụm HPC thường không có giao diện đồ họa.

Những lợi ích chính:

- **Tự động hóa:** một lệnh có thể xử lý hàng trăm file cùng lúc.
- **Tái lập kết quả:** lệnh được lưu thành script, người khác (hoặc chính bạn sau sáu tháng) có thể chạy lại.
- **Làm việc từ xa:** điều khiển máy chủ có nhiều CPU và bộ nhớ hơn laptop.

**Mở terminal**

| Hệ điều hành | Cách mở | Ghi chú |
| --- | --- | --- |
| macOS | Ứng dụng Terminal (Applications > Utilities) hoặc iTerm2 | Shell mặc định là zsh |
| Linux | Ứng dụng Terminal (Ctrl+Alt+T trên Ubuntu) | Shell mặc định thường là bash |
| Windows | Cài WSL2 bằng `wsl --install` trong PowerShell (quyền admin), sau đó mở Ubuntu | Nên dùng WSL thay cho Command Prompt vì hầu hết công cụ sinh tin chạy trên Linux |

**Cấu trúc một lệnh:** `lệnh [tùy chọn] [đối số]`, ví dụ `ls -lh data/`. Khi chưa chắc một lệnh làm gì, có thể xem hướng dẫn bằng `man ls` hoặc `ls --help`, hoặc dán lệnh vào [explainshell.com](https://explainshell.com) để được giải thích từng phần.

## 2. Các lệnh cơ bản

Khoảng mười lệnh dưới đây đủ cho phần lớn thao tác hằng ngày với file và thư mục.

| Lệnh | Ý nghĩa | Ví dụ |
| --- | --- | --- |
| `pwd` | Cho biết bạn đang ở thư mục nào | `pwd` |
| `ls` | Liệt kê file; `-l` chi tiết, `-h` kích thước dễ đọc, `-a` cả file ẩn | `ls -lh data/` |
| `cd` | Chuyển thư mục | `cd ~/projects`, `cd ..` |
| `mkdir -p` | Tạo thư mục (kể cả các thư mục cha) | `mkdir -p data/raw` |
| `cp` | Sao chép; `-r` cho thư mục | `cp -r scripts/ backup/` |
| `mv` | Di chuyển hoặc đổi tên | `mv old.txt new.txt` |
| `rm` | Xóa file; `-r` cho thư mục | `rm temp.txt` |
| `cat`, `less` | Xem nội dung file | `less notes.txt` |
| `chmod` | Đổi quyền truy cập | `chmod +x run.sh` |
| `history` | Xem các lệnh đã gõ | `history \| tail -20` |

**Đường dẫn**

- **Tuyệt đối:** bắt đầu từ gốc `/`, ví dụ `/home/chau/project/data`. Luôn đúng bất kể bạn đang ở đâu.
- **Tương đối:** tính từ thư mục hiện tại, ví dụ `data/raw` hoặc `../results`.
- Ký hiệu đặc biệt: `~` là thư mục home, `.` là thư mục hiện tại, `..` là thư mục cha.
- Tránh dấu cách trong tên file; nếu bắt buộc phải có thì đặt trong ngoặc kép: `cd "My Data"`.

**Phím tắt giúp tiết kiệm thời gian**

- `Tab`: tự hoàn thành tên lệnh và tên file (bấm hai lần để xem các lựa chọn). Dùng Tab cũng giúp tránh gõ sai đường dẫn.
- `↑` / `↓`: duyệt lại các lệnh trước; `Ctrl+R`: tìm trong lịch sử lệnh.
- `Ctrl+C`: dừng lệnh đang chạy; `Ctrl+L`: xóa màn hình.
- `Ctrl+A` / `Ctrl+E`: về đầu hoặc cuối dòng lệnh.

**Lưu ý về `rm`:** dòng lệnh không có thùng rác, file bị xóa thường không khôi phục được. Phần 7 có một vài thói quen giúp xóa an toàn hơn.

## 3. Làm việc với file dữ liệu

File dữ liệu sinh học (FASTQ, BAM, GTF, bảng đếm) thường quá lớn để mở bằng Excel hay trình soạn thảo; các lệnh dưới đây đọc file theo dòng nên xử lý được file hàng chục GB.

| Lệnh | Dùng để | Ví dụ |
| --- | --- | --- |
| `head`, `tail` | Xem vài dòng đầu hoặc cuối | `head -n 8 sample.fastq` |
| `less -S` | Xem file lớn, không xuống dòng (thoát bằng `q`) | `less -S counts.tsv` |
| `zcat`, `zless` | Đọc file `.gz` mà không cần giải nén | `zcat reads.fastq.gz \| head` |
| `wc -l` | Đếm số dòng | `wc -l genes.bed` |
| `grep` | Lọc dòng chứa một mẫu; `-v` để loại, `-c` để đếm | `grep -c ">" genome.fa` |
| `cut` | Lấy một số cột (mặc định phân tách bằng tab) | `cut -f1,3 table.tsv` |
| `sort`, `uniq -c` | Sắp xếp và đếm giá trị lặp | `sort \| uniq -c` |
| `awk` | Lọc và tính toán theo cột | `awk '$5 > 30' calls.tsv` |

**Pipe và chuyển hướng output**

- `|` (pipe) đưa output của lệnh trước làm input cho lệnh sau, giúp ghép nhiều lệnh nhỏ thành một chuỗi xử lý.
- `>` ghi output ra file (ghi đè), `>>` ghi nối thêm, `2>` ghi thông báo lỗi.

**Một vài ví dụ thực tế**

```bash
# Đếm số read trong file FASTQ nén (mỗi read chiếm 4 dòng)
echo $(( $(zcat sample_01.fastq.gz | wc -l) / 4 ))

# Đếm số feature theo loại (cột 3) trong file GTF
grep -v "^#" genes.gtf | cut -f3 | sort | uniq -c | sort -nr

# Chỉ giữ các dòng gene và lưu ra file mới
awk -F'\t' '$3 == "gene"' genes.gtf > genes_only.gtf

# Lưu output và lỗi của một công cụ vào file log
fastqc sample_01.fastq.gz -o results/qc > logs/fastqc.log 2>&1
```

Nên thử chuỗi lệnh trên vài dòng đầu (`head -n 1000 file > test.txt`) trước khi chạy trên toàn bộ dữ liệu; lỗi sẽ lộ ra nhanh hơn và tốn ít thời gian hơn.

## 4. Cài đặt công cụ

Với người mới, cách an toàn nhất thường là cài công cụ qua một trình quản lý gói (conda/mamba) vào thư mục của riêng mình, không dùng `sudo` và không cài thẳng vào hệ thống.

| Cách cài | Khi nào nên dùng | Ghi chú |
| --- | --- | --- |
| conda / mamba (qua Miniforge) | Laptop, server của lab, đa số công cụ sinh tin | Bioconda có hàng nghìn công cụ đã đóng gói sẵn |
| `pip` trong môi trường ảo | Thư viện Python thuần | Luôn chạy bên trong một môi trường, không cài vào Python hệ thống |
| `module load` | Cụm HPC có sẵn phần mềm | Xem bằng `module avail` hoặc `module spider <tên>` |
| Container (Apptainer, Docker) | Cần môi trường giống hệt nhau giữa các máy | Apptainer phổ biến trên HPC vì không cần quyền root |

**Cài Miniforge (conda + mamba)**

Miniforge dùng kênh conda-forge làm mặc định. Nhiều lab chọn Miniforge thay cho Anaconda vì Anaconda có điều khoản giấy phép riêng đối với một số tổ chức.&#32;

(uname prints system info. $(uname) gives the OS (Linux or Darwin for macOS), $(uname -m) the CPU type (x86\_64, arm64). The $( ) inserts that output, so the command downloads the installer matching your machine.)

```bash
# Tải và chạy bộ cài (Linux, macOS, WSL)
curl -L -O "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
bash Miniforge3-$(uname)-$(uname -m).sh

# Mở terminal mới rồi kiểm tra
conda --version
mamba --version
```

**Cấu hình kênh Bioconda (làm một lần)**

```bash
conda config --add channels bioconda
conda config --add channels conda-forge
conda config --set channel_priority strict
```

Thứ tự kênh này giúp conda ưu tiên conda-forge và tránh xung đột phiên bản. `mamba` dùng cùng cú pháp với `conda` nhưng giải phụ thuộc nhanh hơn, nên có thể thay `conda install` bằng `mamba install`.

**Một vài lưu ý**

- Không cài công cụ vào môi trường `base`; hãy giữ `base` gọn và tạo môi trường riêng cho từng project (phần 5).
- Trên Mac chip Apple Silicon, một số package Bioconda có thể chưa có bản arm64. Khi gặp lỗi, có thể tạo môi trường với `CONDA_SUBDIR=osx-64` để dùng bản Intel qua Rosetta.
- Một số cụm HPC, ví dụ của Digital Research Alliance of Canada, không khuyến khích dùng conda và gợi ý dùng module, virtualenv hoặc Apptainer. Nên đọc tài liệu của cụm trước khi cài.
- Tìm package trước khi cài: `mamba search samtools` hoặc tra trên [bioconda.github.io](https://bioconda.github.io/).

## 5. Tạo và quản lý môi trường

Mỗi project nên có một môi trường riêng, được mô tả trong file `environment.yml`, để các phiên bản công cụ không xung đột nhau và người khác có thể dựng lại đúng môi trường đó.

**Vòng đời của một môi trường conda**

```bash
# 1. Tạo môi trường với các công cụ cần thiết (nên ghi rõ phiên bản)
mamba create -n rnaseq python=3.11 samtools=1.20 star fastqc

# 2. Kích hoạt và kiểm tra
conda activate rnaseq
samtools --version

# 3. Cài thêm khi cần
mamba install -n rnaseq multiqc

# 4. Xuất cấu hình để chia sẻ hoặc lưu vào Git
conda env export --from-history > environment.yml

# 5. Thoát, liệt kê, xóa
conda deactivate
conda env list
conda env remove -n rnaseq
```

**Dựng môi trường từ file**

Cách gọn hơn là viết `environment.yml` trước rồi tạo môi trường từ file đó:

```yaml
name: rnaseq
channels:
  - conda-forge
  - bioconda
dependencies:
  - python=3.11
  - samtools=1.20
  - star
  - fastqc
  - pip
  - pip:
      - pandas
```

```bash
mamba env create -f environment.yml    # tạo mới
mamba env update -f environment.yml    # cập nhật sau khi sửa file
```

**Môi trường ảo cho project chỉ dùng Python**

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
pip freeze > requirements.txt          # ghi lại phiên bản đang dùng
```

**Gợi ý**

- Đặt tên môi trường theo project hoặc mục đích (`rnaseq`, `te-analysis`), không đặt `test` hay `env1`.
- `--from-history` chỉ ghi các package bạn yêu cầu, nên dễ dùng lại trên hệ điều hành khác. Nếu cần tái lập chính xác trên cùng loại máy, có thể lưu thêm `conda list --explicit > spec-file.txt`.
- Hạn chế trộn `pip` và `conda` cho cùng một package. Nếu cần `pip`, hãy cài các package conda trước rồi mới dùng `pip`.
- Khi môi trường bị lỗi khó sửa, xóa đi và tạo lại từ `environment.yml` thường nhanh hơn là cố vá.

## 6. Tổ chức project và dữ liệu

Một cấu trúc thư mục nhất quán, cùng nguyên tắc dữ liệu gốc chỉ đọc, giúp bạn và người khác dễ tìm file và biết mỗi kết quả đến từ đâu.

**Cấu trúc gợi ý**

```
ten-project/
├── README.md          # mục đích, nguồn dữ liệu, cách chạy lại
├── environment.yml    # môi trường conda của project
├── data/
│   ├── raw/           # dữ liệu gốc, chỉ đọc
│   └── processed/     # dữ liệu trung gian, tạo lại được
├── scripts/           # code xử lý, đánh số theo thứ tự chạy
├── results/           # bảng, hình, báo cáo
├── logs/              # log của các lần chạy
└── docs/              # ghi chú, sổ tay thí nghiệm
```

&#91;embedded content: luồng dữ liệu trong một project · 3 thư mục dữ liệu, 1 lớp code\]

Dữ liệu chỉ đi theo chiều mũi tên: script đọc từ thư mục trước và ghi vào thư mục sau, nên `data/processed/` và `results/` luôn có thể tạo lại từ `data/raw/` cộng với code.

**Đặt tên file**

- Không dùng dấu cách, dấu tiếng Việt hay ký tự đặc biệt; chỉ dùng chữ thường, số, `_` và `-`.
- Ghi ngày theo dạng `YYYY-MM-DD` (ví dụ `2026-10-29_qc_report.html`) để file tự sắp xếp đúng thứ tự.
- Đánh số có số 0 ở đầu: `sample_01`, `sample_02`... `sample_10`, để `ls` không xếp `sample_10` trước `sample_2`.
- Đánh số script theo thứ tự chạy: `01_qc.sh`, `02_align.sh`, `03_count.R`.

**Bảo vệ dữ liệu gốc**

```bash
# Tạo checksum ngay khi nhận dữ liệu
cd data/raw
md5sum *.fastq.gz > checksums.md5

# Bỏ quyền ghi để tránh sửa hoặc xóa nhầm
chmod -R a-w .
```

**README tối thiểu** nên trả lời: project này làm gì, dữ liệu lấy từ đâu và khi nào, chạy lại kết quả bằng những lệnh nào.

## 7. Best practices khi làm việc với dữ liệu

Phần lớn lỗi trong phân tích dữ liệu không đến từ thuật toán mà từ những thao tác thủ công không được ghi lại; các thói quen dưới đây nhằm giảm loại lỗi đó.

**Viết script thay vì gõ lệnh rời rạc**

Mọi bước có ảnh hưởng tới kết quả nên nằm trong một script. Đầu mỗi script bash nên có:

```bash
#!/usr/bin/env bash
set -euo pipefail   # dừng ngay khi có lỗi, khi dùng biến chưa khai báo, hoặc khi một lệnh trong pipe lỗi

# Mục đích: căn chỉnh read RNA-seq với STAR
# Input:   data/raw/*.fastq.gz
# Output:  data/processed/bam/
```

**Kiểm tra dữ liệu sau khi tải hoặc chuyển**

```bash
md5sum -c checksums.md5    # mọi dòng phải báo OK
```

**Ghi log và phiên bản công cụ**

- Lưu output và lỗi của mỗi lần chạy: `bash scripts/02_align.sh > logs/02_align.log 2>&1`.
- Ghi lại phiên bản đã dùng (`samtools --version`, `conda env export`), vì kết quả có thể thay đổi giữa các phiên bản.

**Dùng Git cho code, không cho dữ liệu lớn**

- Theo dõi `scripts/`, `README.md`, `environment.yml` bằng Git; commit thường xuyên với thông điệp rõ ràng.
- Thêm `data/`, `results/` và `logs/` vào `.gitignore`. File lớn nên lưu ở kho dữ liệu của lab hoặc cụm HPC.

**Giữ lệnh chạy khi mất kết nối**

Khi chạy lâu trên server, mở một phiên `tmux` để lệnh tiếp tục chạy dù mạng bị ngắt:

```bash
tmux new -s align     # tạo phiên mới
# Ctrl+B rồi D để thoát ra mà lệnh vẫn chạy
tmux attach -t align  # quay lại phiên
```

**Xóa an toàn**

- Chạy `ls` với cùng mẫu trước khi `rm`: xem `ls *.bam` liệt kê gì rồi mới `rm *.bam`.
- Dùng `rm -i` khi chưa chắc, và cẩn thận với `rm -rf`, nhất là khi đường dẫn chứa biến.
- Giữ ít nhất một bản sao lưu của dữ liệu gốc ở nơi khác.

**Một vài thói quen khác**

- Giữ file ở dạng nén (`.fastq.gz`, `.bam`); phần lớn công cụ đọc trực tiếp được.
- Tránh mở hoặc lưu lại bảng gene bằng Excel: Excel có thể tự đổi một số tên gene thành ngày tháng.
- Không chạy tác vụ nặng trên login node của HPC (phần 8).
- Khi chuỗi xử lý có nhiều bước và nhiều mẫu, có thể cân nhắc workflow manager như Nextflow hoặc Snakemake để quản lý các bước, chạy lại phần bị lỗi và ghi log tự động.

## 8. Làm việc trên server và HPC

Trên cụm HPC, bạn đăng nhập vào login node để chuẩn bị và gửi job, còn việc tính toán nặng chạy trên compute node thông qua trình lập lịch (thường là Slurm).

**Kết nối bằng SSH**

```bash
# Tạo khóa SSH (làm một lần trên máy của bạn)
ssh-keygen -t ed25519

# Đăng nhập
ssh username@cluster.example.org
```

Để khỏi gõ địa chỉ dài, có thể khai báo trong `~/.ssh/config`:

```
Host hpc
    HostName cluster.example.org
    User username
```

Sau đó chỉ cần `ssh hpc`. Cách thêm khóa công khai vào tài khoản khác nhau giữa các cụm (`ssh-copy-id` hoặc qua trang quản lý tài khoản), nên xem hướng dẫn của cụm.

**Chuyển dữ liệu với rsync**

```bash
# Từ máy bạn lên server; -a giữ thuộc tính, -v hiện chi tiết, -P hiện tiến độ và cho phép tiếp tục khi bị ngắt
rsync -avP data/raw/ hpc:~/projects/rnaseq/data/raw/
```

Dấu `/` ở cuối thư mục nguồn có ý nghĩa: `data/raw/` chép nội dung bên trong, còn `data/raw` chép cả thư mục. Sau khi chuyển, nên chạy lại `md5sum -c`.

**Gửi job với Slurm**

```bash
#!/usr/bin/env bash
#SBATCH --job-name=star_align
#SBATCH --account=def-xxxx          # tài khoản của lab
#SBATCH --time=04:00:00
#SBATCH --cpus-per-task=8
#SBATCH --mem=32G
#SBATCH --output=logs/%x_%j.out     # %x tên job, %j số job

set -euo pipefail
module load star                    # tên và phiên bản: xem bằng module spider star

STAR --runThreadN "$SLURM_CPUS_PER_TASK" ...
```

| Lệnh | Dùng để |
| --- | --- |
| `sbatch job.sh` | Gửi job |
| `squeue -u $USER` | Xem các job đang chờ hoặc đang chạy |
| `scancel <job_id>` | Hủy job |
| `sacct -j <job_id>` | Xem thông tin job đã chạy (thời gian, bộ nhớ) |
| `salloc --time=1:00:00 --cpus-per-task=4 --mem=8G` | Xin một phiên tương tác để thử lệnh |

**Một vài lưu ý**

- Xin tài nguyên vừa đủ: job xin quá nhiều thường phải chờ lâu hơn. Có thể chạy thử trên một mẫu rồi xem `sacct` để ước lượng.
- Các cụm thường chia nhiều vùng lưu trữ (home, project, scratch). Scratch nhanh nhưng có thể bị xóa định kỳ, nên không để dữ liệu duy nhất ở đó.

## 9. Nếu dùng R

Với R, nguyên tắc vẫn giống phần 4 và 5: cài R theo một cách nhất quán, quản lý package riêng cho từng project, và chạy phân tích bằng script để có thể tái lập.

**Chọn cách cài R**

| Cách cài | Khi nào nên dùng | Quản lý package |
| --- | --- | --- |
| conda / mamba | Muốn R chung môi trường với công cụ sinh tin khác (samtools, STAR...) | Cài qua conda: `r-<tên>` cho CRAN, `bioconductor-<tên>` cho Bioconductor |
| R từ CRAN, hoặc qua [rig](https://github.com/r-lib/rig) | Laptop, dùng RStudio hoặc Positron | renv cho từng project |
| `module load r` | Cụm HPC | Thư viện cá nhân kèm renv |

Mỗi project nên theo một cách. Trộn R của conda với R hệ thống, hoặc vừa cài package bằng conda vừa bằng `install.packages()`, dễ gây lỗi khó tìm nguyên nhân.

**Cách 1: R trong môi trường conda**

```bash
mamba create -n r-de -c conda-forge -c bioconda r-base r-tidyverse r-here bioconductor-deseq2
conda activate r-de
R --version
```

Sau đó xuất `environment.yml` như phần 5 để lưu cấu hình.

**Cách 2: R hệ thống kèm renv**

renv tạo thư viện package riêng trong thư mục project và ghi phiên bản của từng package vào file `renv.lock`.

```r
install.packages("renv")          # làm một lần
renv::init(bioconductor = TRUE)   # chạy trong thư mục gốc của project
renv::install("tidyverse")        # package CRAN
renv::install("bioc::DESeq2")     # package Bioconductor
renv::snapshot()                  # ghi phiên bản vào renv.lock
renv::restore()                   # trên máy khác: cài lại đúng các phiên bản đã ghi
```

Đưa `renv.lock` vào Git; thư mục `renv/library/` thì không cần (renv tự loại nó qua `.gitignore`). Nếu cần nhiều phiên bản R trên cùng laptop, `rig add release`, `rig list` và `rig default` giúp cài và chuyển đổi giữa các phiên bản.

**Chạy R từ dòng lệnh**

```bash
# Chạy một script, truyền đường dẫn input và output
Rscript scripts/03_deseq2.R data/processed/counts.tsv results/de

# Render báo cáo
Rscript -e 'rmarkdown::render("docs/report.Rmd")'
quarto render docs/report.qmd
```

Script nhận đối số từ dòng lệnh thay vì gán cứng đường dẫn:

```r
#!/usr/bin/env Rscript
args <- commandArgs(trailingOnly = TRUE)
counts_file <- args[1]
out_dir <- args[2]
dir.create(out_dir, recursive = TRUE, showWarnings = FALSE)

suppressPackageStartupMessages({
  library(DESeq2)
  library(readr)
})

set.seed(42)
counts <- read_tsv(counts_file)
# ... phân tích ...

# Lưu lại phiên bản R và package đã dùng
writeLines(capture.output(sessionInfo()), file.path(out_dir, "sessionInfo.txt"))
```

**Chạy R trong job Slurm**

Trong script `sbatch` (phần 8), thay dòng chạy công cụ bằng:

```bash
module load r                        # tên và phiên bản: xem bằng module spider r
Rscript scripts/03_deseq2.R data/processed/counts.tsv results/de
```

Nếu dùng môi trường conda trong job, có thể cần chạy `eval "$(conda shell.bash hook)"` trước `conda activate r-de`.

**Thói quen tốt với R**

- Dùng đường dẫn tương đối từ gốc project (package `here` hỗ trợ việc này), tránh `setwd("/Users/ten/...")`.
- Không lưu và tự nạp lại workspace: chạy `R --no-save`, và trong RStudio tắt tùy chọn khôi phục `.RData` khi khởi động. Khởi động lại R thường xuyên giúp chắc rằng script chạy được từ đầu.
- Với RStudio, mở project qua file `.Rproj` để thư mục làm việc luôn là gốc project.
- Đặt `set.seed()` khi có bước ngẫu nhiên, và lưu `sessionInfo()` cùng kết quả.
- Phân tích khám phá có thể làm trong RStudio hay notebook, nhưng các bước tạo kết quả cuối nên được chuyển thành script trong `scripts/`.

Tài liệu thêm: [renv](https://rstudio.github.io/renv/), [cài đặt Bioconductor](https://bioconductor.org/install/), [R trên cụm của Alliance](https://docs.alliancecan.ca/wiki/R).

## 10. Tra cứu nhanh và tài liệu học thêm

| Tôi muốn... | Lệnh |
| --- | --- |
| Xem dung lượng một thư mục | `du -sh data/` |
| Xem dung lượng ổ đĩa còn trống | `df -h` |
| Tìm file theo tên | `find . -name "*.bam"` |
| Tìm một chuỗi trong mọi file code | `grep -rn "read_csv" scripts/` |
| Đếm số file trong thư mục | `ls data/raw/*.fastq.gz \| wc -l` |
| Xem tiến trình và mức dùng CPU, RAM | `top` hoặc `htop` |
| Nén và giải nén | `gzip file`, `gunzip file.gz`, `tar -czf out.tar.gz folder/` |
| Tải file từ internet | `wget <url>` hoặc `curl -O <url>` |
| Chạy cùng một lệnh cho nhiều file | `for f in data/raw/*.fastq.gz; do fastqc "$f" -o results/qc; done` |
| Xem môi trường conda đang dùng | `conda info --envs` |

**Tài liệu học thêm**

- [The Unix Shell, Software Carpentry](https://swcarpentry.github.io/shell-novice/): bài học từng bước cho người mới.
- [Bioconda](https://bioconda.github.io/): hướng dẫn cài đặt và tra cứu package.
- [Tài liệu conda](https://docs.conda.io/projects/conda/en/latest/user-guide/index.html): quản lý môi trường chi tiết.
- [Hướng dẫn về Anaconda trên cụm của Alliance](https://docs.alliancecan.ca/wiki/Anaconda/en): lý do và các lựa chọn thay thế trên HPC ở Canada.
- [explainshell.com](https://explainshell.com): giải thích từng phần của một lệnh.
- Wilson và cộng sự (2017), [Good enough practices in scientific computing](https://doi.org/10.1371/journal.pcbi.1005510), PLOS Computational Biology.
- Noble (2009), [A quick guide to organizing computational biology projects](https://doi.org/10.1371/journal.pcbi.1000424), PLOS Computational Biology.
