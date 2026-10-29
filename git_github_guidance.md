# Git và GitHub cho xử lý dữ liệu sinh học

Oct 29, 2026 · @Chau Duong

## Vì sao dùng Git khi xử lý dữ liệu sinh học

Trong tin sinh học, Git quản lý phần "công thức" của phân tích (mã, cấu hình, môi trường, tài liệu), còn dữ liệu thô và kết quả lớn nằm ở nơi khác. Nhờ vậy, mỗi hình hay bảng trong bài báo có thể truy ngược về đúng phiên bản mã đã tạo ra nó.

Git là công cụ quản lý phiên bản chạy trên máy cá nhân hoặc cụm HPC; GitHub là dịch vụ web lưu bản sao của repo Git (repository) để chia sẻ, review và phát hành.

| Đưa vào Git | Để ngoài Git |
| --- | --- |
| Script R, Python, Bash | FASTQ, BAM, CRAM, VCF, tệp ảnh thô |
| Pipeline Nextflow, Snakemake và tệp cấu hình | Thư mục `work/`, `.snakemake/`, kết quả trung gian |
| Samplesheet không chứa thông tin định danh | Metadata lâm sàng, mã bệnh nhân |
| `environment.yml`, `renv.lock`, `Dockerfile` | Genome tham chiếu, chỉ mục STAR, BWA |
| README, ghi chú phương pháp, notebook đã xóa output | Ma trận đếm lớn, đối tượng `.rds`, `.h5ad` dung lượng lớn |
| Bộ dữ liệu test rất nhỏ (vài trăm read) | Token, mật khẩu, khóa API |

Một vài lợi ích cụ thể:

- Biết chính xác phiên bản pipeline và tham số đã dùng cho từng lần chạy.
- Thử aligner, tham số hay phiên bản genome khác trên một nhánh riêng mà không làm hỏng phân tích chính.
- Đồng bộ mã giữa laptop và cụm HPC mà không phải sao chép thủ công.
- Đáp ứng yêu cầu chia sẻ mã của nhiều tạp chí và quỹ tài trợ khi công bố.

## Cài đặt và cấu hình ban đầu

Chỉ cần làm một lần trên mỗi máy: cài Git, khai báo tên và email, rồi kết nối với GitHub bằng SSH key.

**Cài Git**

| Hệ điều hành | Cách cài |
| --- | --- |
| macOS | `xcode-select --install` hoặc `brew install git` |
| Ubuntu, Debian | `sudo apt install git` |
| Windows | Tải Git for Windows tại [git-scm.com](https://git-scm.com) (kèm Git Bash) |
| Cụm HPC | Thường có sẵn; thử `module load git` nếu cần bản mới hơn |

Kiểm tra bằng `git --version`.

**Khai báo danh tính**

Tên và email sẽ gắn vào mọi commit. Email nên trùng với email tài khoản GitHub để commit được liên kết đúng.

```bash
 git config --global user.name "cduong11"
git config --global user.email "cduong11@example.com"
git config --global init.defaultBranch main
git config --global core.editor "nano"   # hoặc "code --wait", "vim"
git config --list                         # xem lại cấu hình
```

**Kết nối GitHub bằng SSH key**

GitHub không còn chấp nhận mật khẩu tài khoản khi push qua HTTPS, nên SSH key (hoặc personal access token) gần như là bắt buộc.

1. Tạo key: `ssh-keygen -t ed25519 -C "cduong@example.com"` và nhấn Enter để dùng đường dẫn mặc định.
2. Sao chép khóa công khai: `cat ~/.ssh/id_ed25519.pub`.
3. Trên GitHub, vào Settings, chọn SSH and GPG keys, bấm New SSH key và dán nội dung vào.
4. Kiểm tra: `ssh -T git@github.com`. Nếu thấy lời chào kèm tên tài khoản là đã thành công.

Khi clone, hãy dùng địa chỉ dạng `git@github.com:user/repo.git` thay vì `https://...` để SSH key được sử dụng.

**Trên cụm HPC**

Cụm tính toán là một máy riêng, nên cần cấu hình Git và tạo SSH key riêng ở đó (lặp lại các bước trên sau khi đăng nhập vào cụm). Một số điểm thường gặp:

- Phiên bản Git hệ thống đôi khi khá cũ; thử `module avail git` rồi `module load git` để dùng bản mới hơn.
- Đặt repo mã trong thư mục home hoặc project (có sao lưu), còn dữ liệu và thư mục `work/` của pipeline đặt ở scratch hoặc vùng lưu trữ dự án.
- Một số nút tính toán không có internet; nên `git pull` trên nút đăng nhập trước khi gửi job.
- Nếu cụm chặn cổng 22, có thể dùng SSH qua cổng 443 của GitHub (`ssh.github.com`) theo hướng dẫn trong GitHub Docs.

## Khái niệm cốt lõi

Một thay đổi đi qua ba vùng trên máy của bạn trước khi lên GitHub; hiểu được các vùng này thì phần lớn lệnh Git trở nên dễ đoán.

&#91;embedded content: Bốn vùng của Git và các lệnh di chuyển giữa chúng\]

Sửa tệp xảy ra ở vùng làm việc; `git add` chọn những thay đổi muốn lưu, `git commit` ghi chúng vào lịch sử, còn `git push` gửi lịch sử đó lên GitHub.

| Thuật ngữ | Ý nghĩa |
| --- | --- |
| repository (repo) | Thư mục dự án cùng toàn bộ lịch sử, lưu trong thư mục ẩn `.git` |
| Commit | Một bản chụp các tệp tại một thời điểm, có mã định danh (hash), tác giả, thời gian và message |
| Branch (nhánh) | Một dòng phát triển riêng; thực chất là con trỏ trỏ tới một commit |
| `main` | Nhánh mặc định, thường là phiên bản ổn định |
| HEAD | Con trỏ cho biết bạn đang đứng ở commit hoặc nhánh nào |
| Remote | Bản sao của repo ở nơi khác, thường trên GitHub |
| `origin` | Tên mặc định của remote khi bạn clone |
| Merge | Gộp lịch sử của hai nhánh |
| Pull request (PR) | Đề xuất gộp nhánh trên GitHub, kèm thảo luận và review |

## Cấu trúc một dự án phân tích

Một cấu trúc thư mục cố định, trong đó `data/` và `results/` bị Git bỏ qua, giúp tách mã khỏi dữ liệu ngay từ đầu.

```text
rnaseq-te-analysis/
├── README.md            # mục tiêu, cách chạy, dữ liệu lấy ở đâu
├── workflows/           # pipeline Nextflow hoặc Snakemake
├── conf/                # cấu hình theo cụm, theo profile
├── scripts/             # script R, Python, Bash
├── notebooks/           # khám phá dữ liệu, output đã xóa
├── envs/                # environment.yml, renv.lock, Dockerfile
├── assets/              # samplesheet mẫu, tệp nhỏ cố định
├── tests/data/          # bộ dữ liệu test rất nhỏ
├── data/                # (bỏ qua) liên kết tới dữ liệu thô
└── results/             # (bỏ qua) kết quả
```

Thư mục `data/` thường chỉ chứa liên kết mềm (`ln -s /project/raw/fastq data/fastq`) để đường dẫn trong mã ngắn gọn mà dữ liệu vẫn nằm trên vùng lưu trữ chung.

**.gitignore cho dữ liệu sinh học**

```text
# Dữ liệu thô và căn chỉnh (dấu / ở đầu: chỉ thư mục ở gốc kho)
/data/
*.fastq
*.fastq.gz
*.fq.gz
*.bam
*.bai
*.cram
*.crai
*.sam
*.vcf.gz
*.tbi
*.bigwig
*.bw

# Tham chiếu và chỉ mục
*.fa
*.fa.gz
*.fasta
*.fai
*.2bit
/references/

# Kết quả và đối tượng phân tích lớn
/results/
*.rds
*.h5ad
*.loom

# Nextflow, Snakemake
work/
.nextflow*
.snakemake/

# R, Python, hệ thống
.Rhistory
.RData
.Rproj.user/
__pycache__/
.ipynb_checkpoints/
.DS_Store

# Bí mật
.env
*.pem

# Ngoại lệ: dữ liệu test nhỏ
!tests/data/*.fastq.gz
```

Dòng `!` ở cuối giữ lại các tệp test nhỏ dù chúng trùng mẫu `*.fastq.gz`; ngoại lệ chỉ có tác dụng khi thư mục cha của tệp không bị bỏ qua, vì vậy mới cần viết `/data/` thay vì `data/`. Một tệp đã commit trước đó sẽ không tự biến mất khi thêm vào `.gitignore`; cần chạy `git rm --cached ten-tep`. Để kiểm tra vì sao một tệp bị bỏ qua, dùng `git check-ignore -v ten-tep`.

## Các lệnh hằng ngày qua một ví dụ RNA-seq

Phần lớn công việc là một vòng lặp: sửa script hoặc cấu hình, `git add`, `git commit`, `git push`, rồi `git pull`.

**Bắt đầu dự án**

```bash
# Trên laptop: tạo repo mới cho phân tích
mkdir rnaseq-te-analysis && cd rnaseq-te-analysis
git init
# tạo README.md, .gitignore, cấu trúc thư mục như phần trên
git add README.md .gitignore
git commit -m "Initialize analysis project structure"

# Tạo repo trống trên GitHub (không tích README), rồi kết nối và push lần đầu
# hoặc git@github.com:cduong11/rnaseq-te-analysis.git
git remote add origin https://github.com/cduong11/rnaseq-te-analysis.git.  

git push -u origin main

# Trên cụm HPC: lấy repo về sau khi đã push lên GitHub
git clone git@github.com:cduong11/rnaseq-te-analysis.git
```

**Vòng làm việc thông thường**

```bash
git status                               # tệp nào đã sửa, đã stage
git diff conf/params.config              # xem thay đổi cụ thể
git add scripts/deseq2_viral_load.R conf/params.config
git diff --staged                        # kiểm tra trước khi commit
git commit -m "Add viral load covariate to DESeq2 design"
git push
```

Nên `git add` từng tệp cụ thể thay vì `git add .` cho đến khi `.gitignore` đã chắc chắn; `git add .` dễ vô tình push một tệp BAM hay bảng metadata.

**Đồng bộ laptop và cụm HPC (login node)**

```bash
# Trên cụm, trước khi gửi job
cd ~/projects/rnaseq-te-analysis
git pull
git log -1 --oneline                     # ghi lại commit sẽ dùng để chạy
sbatch run_pipeline.sh
```

Một quy ước hữu ích là chỉ sửa mã ở một nơi (thường là laptop), còn cụm chỉ `git pull`. Nếu buộc phải sửa nhanh trên cụm, hãy commit và push ngay để tránh xung đột về sau.

**Xem lịch sử**

```bash
git log --oneline --graph --all          # lịch sử gọn, có nhánh
git log -p -- conf/params.config         # tham số đã thay đổi thế nào qua thời gian
git log --since="2 weeks ago" -- scripts/
git show a1b2c3d                         # nội dung một commit
git blame scripts/filter_te.R            # dòng nào được sửa ở commit nào
```

Khi một kết quả cũ trông khác kết quả mới, `git diff <commit-cu> <commit-moi> -- scripts/ conf/` thường là cách nhanh nhất để tìm ra thay đổi nào gây ra khác biệt.

## Nhánh để thử nghiệm phân tích

Nhánh phù hợp với kiểu làm việc thử và sai của phân tích dữ liệu: `main` giữ phiên bản đang dùng cho kết quả chính, còn mỗi thử nghiệm (aligner khác, ngưỡng lọc khác, genome mới) nằm trên một nhánh riêng.

**Tạo và chuyển nhánh**

```bash
git switch -c exp/t2t-reference          # thử genome T2T-CHM13 thay cho GRCh38
# sửa conf/params.config, chạy lại pipeline trên một tập mẫu nhỏ
git add conf/params.config
git commit -m "Switch reference to T2T-CHM13 for TE quantification"
git push -u origin exp/t2t-reference

git switch main                          # quay lại phiên bản chính bất cứ lúc nào
git branch -a                            # liệt kê mọi nhánh, kể cả trên GitHub
```

Kết quả của mỗi nhánh nên ghi ra thư mục riêng (ví dụ `--outdir results/t2t-reference`) để các lần chạy không ghi đè lên nhau.

**Gộp khi thử nghiệm thành công**

```bash
git switch main
git pull
git merge exp/t2t-reference
git push
git branch -d exp/t2t-reference          # xóa nhánh đã gộp
```

Nếu thử nghiệm không thành công, có thể giữ nhánh lại làm hồ sơ (ví dụ đổi tên thành `archive/...`) thay vì xóa; reviewer đôi khi hỏi vì sao một lựa chọn bị loại.

**Xử lý xung đột**

Xung đột xảy ra khi hai nhánh cùng sửa một dòng, thường gặp ở tệp cấu hình chung:

```text
<<<<<<< HEAD
params.genome  = "GRCh38"
params.min_mapq = 10
=======
params.genome  = "T2T-CHM13"
params.min_mapq = 10
>>>>>>> exp/t2t-reference
```

1. Mở tệp, giữ giá trị đúng và xóa các dòng `<<<<<<<`, `=======`, `>>>>>>>`.
2. `git add conf/params.config` để đánh dấu đã giải quyết.
3. `git commit` để hoàn tất; muốn hủy việc gộp thì dùng `git merge --abort`.

**Rebase**

`git rebase main` đặt các commit của nhánh lên đầu `main` để lịch sử thẳng hơn. Vì rebase viết lại lịch sử, chỉ nên dùng cho nhánh chưa ai khác kéo về; với nhánh dùng chung trong nhóm, merge thường an toàn hơn.

**Đặt tên nhánh**

Tiền tố cho biết mục đích giúp danh sách nhánh dễ đọc: `exp/` cho thử nghiệm phân tích, `feature/` cho bước pipeline mới, `fix/` cho sửa lỗi, `paper/` cho phiên bản gắn với một bản thảo.

## GitHub cho nhóm nghiên cứu

Với một nhóm tin sinh học, GitHub thường đảm nhận ba việc: nơi review thay đổi trong pipeline qua pull request, nơi chạy kiểm thử tự động trên dữ liệu test nhỏ, và nơi phát hành phiên bản mã đi kèm bài báo.

**Tổ chức repo của nhóm**

- Tạo một GitHub Organization cho phòng thí nghiệm để repo không phụ thuộc vào tài khoản cá nhân khi thành viên rời nhóm.
- repo phân tích trên dữ liệu chưa công bố nên để Private; chuyển sang Public khi nộp bài hoặc khi bài được chấp nhận.
- Sinh viên có thể đăng ký GitHub Education để dùng các tính năng trả phí miễn phí.

**Pull request cho thay đổi trong pipeline**

1. Tạo nhánh từ `main` mới nhất, ví dụ `feature/add-salmon-quant`.
2. Commit, rồi `git push -u origin feature/add-salmon-quant`.
3. Mở pull request, mô tả: bước nào thay đổi, chạy thử trên dữ liệu nào, kết quả có khác bản cũ không.
4. Người review góp ý trực tiếp trên từng dòng; sửa tiếp trên cùng nhánh, PR tự cập nhật.
5. Gộp (thường chọn Squash and merge), xóa nhánh, rồi `git switch main && git pull` trên laptop và cụm.

**Issues để theo dõi vấn đề phân tích**

Issue phù hợp để ghi các việc như "mẫu S12 có tỉ lệ map thấp, cần kiểm tra" hay "thử thêm hiệu chỉnh batch". Viết `Fixes #12` trong commit message hoặc mô tả PR sẽ tự đóng issue số 12 khi gộp. Tránh ghi mã bệnh nhân hay thông tin lâm sàng trong issue, kể cả ở repo Private.

**Dùng và đóng góp cho pipeline cộng đồng**

Các pipeline của [nf-core](https://nf-co.re) đều nằm trên GitHub, nên có thể chạy thẳng một phiên bản cụ thể:

```bash
nextflow run nf-core/rnaseq -r 3.14.0 -profile singularity \
    --input samplesheet.csv --genome GRCh38 --outdir results
```

Khi cần sửa một pipeline không thuộc quyền của mình, fork repo về tài khoản, làm việc trên bản fork, rồi mở PR ngược lại repo gốc:

```bash
git clone git@github.com:ban/rnaseq.git && cd rnaseq
git remote add upstream git@github.com:nf-core/rnaseq.git
git fetch upstream
git merge upstream/dev                   # nf-core phát triển trên nhánh dev
```

**Kiểm thử tự động với GitHub Actions**

Một workflow nhỏ trong `.github/workflows/test.yml` có thể chạy pipeline trên bộ dữ liệu test mỗi khi push hoặc mở PR, giúp phát hiện sớm thay đổi làm hỏng pipeline:

```yaml
name: test-pipeline
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: nf-core/setup-nextflow@v2
      - run: nextflow run . -profile test,docker --outdir results
```

Dữ liệu test cần đủ nhỏ để chạy trong vài phút trên máy ảo của GitHub, ví dụ vài nghìn read trên một nhiễm sắc thể.

**Phát hành phiên bản đi kèm bài báo**

| Việc | Cách làm |
| --- | --- |
| Đánh dấu phiên bản | `git tag -a v1.0.0 -m "Manuscript submission"` rồi `git push --tags` |
| Tạo release | Trên GitHub, mục Releases, chọn tag và viết ghi chú thay đổi |
| Cấp DOI | Liên kết repo với [Zenodo](https://zenodo.org); mỗi release được lưu trữ và cấp DOI để trích dẫn |
| Hướng dẫn trích dẫn | Thêm tệp `CITATION.cff` ở gốc kho; GitHub hiển thị nút Cite this repo sitory |
| Giấy phép | Thêm tệp `LICENSE` (ví dụ MIT cho mã) để người khác được phép tái sử dụng |

## Hoàn tác và sửa lỗi

Cách hoàn tác phụ thuộc vào việc commit đã được push hay chưa; riêng trường hợp lỡ commit dữ liệu lớn hoặc nhạy cảm thì cần xử lý cả lịch sử, không chỉ commit mới nhất.

| Tình huống | Lệnh | Ghi chú |
| --- | --- | --- |
| Bỏ thay đổi chưa stage trong một script | `git restore scripts/qc.R` | Mất hẳn thay đổi đó |
| Lỡ `git add` một tệp BAM, chưa commit | `git restore --staged data/S1.bam` | Rồi thêm mẫu vào `.gitignore` |
| Quên thêm tệp hoặc viết sai message ở commit vừa tạo | `git add ...` rồi `git commit --amend` | Chỉ dùng khi chưa push |
| Hủy commit cuối, giữ thay đổi để sửa tiếp | `git reset --soft HEAD~1` | Chỉ dùng khi chưa push |
| Đổi tham số làm hỏng kết quả, đã push | `git revert <commit>` | Tạo commit đảo ngược, lịch sử vẫn ghi lại lần thử |
| Cần chạy lại phân tích bằng mã cũ | `git switch --detach v1.0.0` | Xem mã ở phiên bản cũ; `git switch main` để quay lại |
| Lấy lại một script như ở commit cũ | `git restore --source=<commit> scripts/qc.R` | Không đổi các tệp khác |
| Cất tạm thay đổi dang dở trước khi `git pull` | `git stash`, sau đó `git stash pop` | `git stash list` để xem các bản đã cất |
| Lỡ tay `reset --hard` | `git reflog` rồi `git reset --hard <ma>` | Thường khôi phục được trong vài tuần |

**Lỡ commit dữ liệu lớn hoặc nhạy cảm**

Xóa tệp ở commit mới không đủ: tệp vẫn nằm trong lịch sử và trong mọi bản clone. Nếu chưa push, cách đơn giản nhất là `git reset --soft` về trước commit đó, bỏ tệp ra rồi commit lại. Nếu đã push:

1. Với dữ liệu nhạy cảm (mã bệnh nhân, metadata lâm sàng, token), báo ngay cho người phụ trách dữ liệu và thu hồi token; việc này quan trọng hơn việc dọn lịch sử.
2. Dùng [git-filter-repo ](https://github.com/newren/git-filter-repo ) để gỡ tệp khỏi toàn bộ lịch sử, ví dụ `git filter-repo  --path data/clinical_metadata.csv --invert-paths`.
3. `git push --force` lên mọi nhánh bị ảnh hưởng, rồi yêu cầu mọi người xóa bản clone cũ và clone lại.
4. Với repo trên GitHub, có thể cần liên hệ GitHub Support để xóa bản lưu đệm của các commit cũ.

Các lệnh có `--hard`, `--force` hoặc `filter-repo ` đều viết lại hay xóa dữ liệu; nên chạy `git status` và sao lưu repo trước khi dùng.

## Dữ liệu lớn: để ngoài Git

Git lưu vĩnh viễn mọi phiên bản của mọi tệp trong lịch sử và trong từng bản clone, nên không phù hợp với tệp giải trình tự hàng GB; GitHub cũng chặn tệp trên 100 MB và cảnh báo từ 50 MB.

&#91;embedded content: Mã, dữ liệu và môi trường nằm ở đâu trong một lần chạy pipeline\]

repo Git chỉ cần chứa đủ thông tin để tìm lại dữ liệu và môi trường: đường dẫn, mã truy cập trên repo dữ liệu công khai, tag của image và checksum. Commit hash ghi kèm kết quả là sợi dây nối ba nơi này với nhau.

**Chọn cách quản lý dữ liệu**

| Cách | Phù hợp khi | Lưu ý |
| --- | --- | --- |
| Ghi đường dẫn, mã truy cập và checksum trong README hoặc samplesheet | Dữ liệu cố định trên cụm hoặc trên SRA, ENA, GEO | Đơn giản nhất, thường đủ cho đa số dự án |
| [Git LFS](https://git-lfs.com) | Vài tệp cỡ chục đến vài trăm MB cần đi kèm mã, ví dụ một tệp chú thích TE đã xử lý | GitHub giới hạn dung lượng và băng thông LFS ở gói miễn phí |
| [DVC](https://dvc.org) | Cần quản lý phiên bản cả dữ liệu và kết quả trung gian, lưu trên S3, máy chủ SSH hoặc Google Drive | Thêm một công cụ cần học và duy trì |

**Checksum để chắc dữ liệu không đổi**

```bash
md5sum /project/raw/fastq/*.fastq.gz > assets/fastq_md5.txt   # tệp nhỏ, commit được
md5sum -c assets/fastq_md5.txt                                 # kiểm tra sau khi chuyển hoặc tải lại dữ liệu
```

**Git LFS cơ bản**

```bash
git lfs install
git lfs track "assets/*.gtf.gz"
git add .gitattributes assets/te_annotation.gtf.gz
git commit -m "Track TE annotation with Git LFS"
```

## Dữ liệu nhạy cảm và dữ liệu lâm sàng

Với dữ liệu từ người (đoàn hệ bệnh nhân, dữ liệu kiểm soát truy cập như dbGaP hay EGA), nguyên tắc an toàn nhất là không để bất kỳ dữ liệu cấp cá nhân nào đi vào Git, kể cả trong repo Private.

- Samplesheet trong repo chỉ nên chứa mã mẫu đã được mã hóa và đường dẫn tệp; bảng liên kết mã mẫu với mã bệnh nhân và metadata lâm sàng nằm trên hệ thống được phê duyệt của dự án.
- Kiểm tra cả những nơi dễ bỏ sót: output của notebook, bảng kết quả có cột tuổi, giới tính, ngày lấy mẫu, hình có nhãn mẫu, tệp log.
- Không đặt thông tin đăng nhập (token, mật khẩu cơ sở dữ liệu) trong mã; dùng biến môi trường hoặc tệp `.env` đã nằm trong `.gitignore`.
- Bật secret scanning và push protection của GitHub để chặn việc push nhầm token.
- Đọc kỹ thỏa thuận sử dụng dữ liệu (data use agreement) và quy định đạo đức của dự án; một số thỏa thuận hạn chế cả việc công bố số liệu tổng hợp ở mức nhóm nhỏ.

Khi công bố, một cách phổ biến là để mã ở repo Public và ghi rõ trong README cách xin quyền truy cập dữ liệu gốc qua repo dữ liệu kiểm soát truy cập.

## Tính tái lập

Một kết quả có thể tái lập khi biết đủ ba thứ: phiên bản mã (commit hoặc tag), môi trường phần mềm, và dữ liệu đầu vào. Git trực tiếp giải quyết thứ nhất và lưu được mô tả của hai thứ còn lại.

**Môi trường phần mềm**

| Công cụ | Tệp đưa vào Git |
| --- | --- |
| Conda, Mamba | `environment.yml` (nên ghi rõ phiên bản, ví dụ `star=2.7.11b`) |
| R | `renv.lock` (tạo bằng `renv::snapshot()`) |
| Python | `requirements.txt` hoặc `pyproject.toml` |
| Container | `Dockerfile` hoặc tệp định nghĩa Apptainer; tên image kèm tag hoặc digest trong cấu hình pipeline |

**Chạy pipeline theo phiên bản cố định**

```bash
# Chạy pipeline của nhóm ở đúng tag đã dùng cho bài báo
nextflow run lab/rnaseq-te-pipeline -r v1.0.0 -profile cluster \
    --input samplesheet.csv --outdir results/v1.0.0
```

Với Nextflow, khai báo `manifest` (tên, phiên bản) trong `nextflow.config`; báo cáo của Nextflow và các tệp trong `pipeline_info/` sẽ ghi lại commit đã chạy.

**Ghi commit vào kết quả**

Với script tự viết, có thể lưu commit hash cạnh mỗi kết quả để luôn biết bảng hay hình nào đến từ phiên bản mã nào:

```bash
# trong script chạy phân tích
echo "$(git rev-parse HEAD)" > results/run_info/git_commit.txt
git diff --quiet || echo "Cảnh báo: có thay đổi chưa commit" >&2
```

```r
# trong R
writeLines(system("git rev-parse HEAD", intern = TRUE),
           "results/run_info/git_commit.txt")
sessionInfo()
```

Thói quen chỉ chạy phân tích chính khi `git status` sạch (không có thay đổi chưa commit) giúp commit hash ghi lại phản ánh đúng mã đã chạy.

## Notebook và commit message

Notebook Jupyter lưu cả output (bảng, hình, đôi khi dữ liệu mẫu) bên trong tệp JSON, nên diff khó đọc và dễ làm lộ dữ liệu; xóa output trước khi commit là cách xử lý phổ biến.

**Jupyter**

```bash
pip install nbstripout
nbstripout --install          # tự xóa output mỗi khi commit trong repo này
```

Một lựa chọn khác là [Jupytext](https://jupytext.readthedocs.io), ghép mỗi notebook với một tệp `.py` dạng văn bản để commit tệp đó thay cho `.ipynb`.

**R Markdown và Quarto**

Tệp `.Rmd` và `.qmd` vốn là văn bản nên làm việc tốt với Git. Chỉ commit tệp nguồn; tệp HTML hoặc PDF sinh ra nên nằm trong `.gitignore`, trừ khi cố ý dùng để xuất bản (ví dụ qua GitHub Pages).

**Commit message cho phân tích**

Commit message tốt trả lời câu hỏi "thay đổi gì và vì sao", điều mà vài tháng sau khi viết phần phương pháp thường khó nhớ lại.

| Nên | Không nên |
| --- | --- |
| `Filter TEs with fewer than 10 reads in 20% of samples` | `update filter` |
| `Add sex and batch as covariates in DESeq2 design` | `fix model` |
| `Bump STAR to 2.7.11b to match reference index` | `env changes` |
| `Exclude S12: mapping rate 31%, see #8` | `remove sample` |

Dòng đầu nên ngắn (khoảng 50 ký tự), viết ở dạng mệnh lệnh; nếu cần giải thích thêm, để trống một dòng rồi viết phần thân. Mỗi commit nên gói một thay đổi có ý nghĩa trọn vẹn, ví dụ tách riêng việc đổi tham số lọc với việc sửa hình.

## Bảng tra cứu nhanh

| Nhóm | Lệnh | Tác dụng |
| --- | --- | --- |
| Khởi tạo | `git init` / `git clone <url>` | Tạo repo mới / tải repo về laptop hoặc cụm |
| Theo dõi | `git status` | Trạng thái các tệp |
| Theo dõi | `git diff` / `git diff --staged` | Thay đổi chưa stage / đã stage |
| Theo dõi | `git check-ignore -v <tệp>` | Vì sao một tệp bị bỏ qua |
| Lưu | `git add <tệp>` / `git commit -m "..."` | Stage và tạo commit |
| Đồng bộ | `git pull` / `git push` | Lấy về / đẩy lên GitHub |
| Lịch sử | `git log --oneline --graph --all` | Lịch sử gọn, có nhánh |
| Lịch sử | `git log -p -- conf/params.config` | Tham số thay đổi thế nào qua thời gian |
| Lịch sử | `git rev-parse HEAD` | Commit hiện tại, để ghi vào kết quả |
| Nhánh | `git switch -c exp/<tên>` | Tạo nhánh thử nghiệm |
| Nhánh | `git merge <nhánh>` | Gộp nhánh vào nhánh hiện tại |
| Phiên bản | `git tag -a v1.0.0 -m "..."` + `git push --tags` | Đánh dấu phiên bản dùng cho bài báo |
| Phiên bản | `git switch --detach v1.0.0` | Xem lại mã ở phiên bản cũ |
| Hoàn tác | `git restore --staged <tệp>` | Bỏ stage, ví dụ tệp BAM lỡ add |
| Hoàn tác | `git revert <commit>` | Đảo ngược commit đã push |
| Hoàn tác | `git stash` / `git stash pop` | Cất tạm / lấy lại thay đổi |
| Cứu hộ | `git reflog` | Tìm lại commit tưởng đã mất |
| Dọn lịch sử | `git filter-repo  --path <tệp> --invert-paths` | Gỡ tệp khỏi toàn bộ lịch sử |
| Tệp lớn | `git lfs track "*.rds"` | Theo dõi tệp lớn bằng Git LFS |
| Notebook | `nbstripout --install` | Tự xóa output notebook khi commit |
| Pipeline | `nextflow run <user/repo > -r <tag>` | Chạy pipeline ở phiên bản cố định |

**Tài liệu tham khảo**

- [Pro Git](https://git-scm.com/book/vi/v2): sách miễn phí, có bản tiếng Việt.
- [Software Carpentry: Version Control with Git](https://swcarpentry.github.io/git-novice/): bài học dành cho nhà nghiên cứu.
- [nf-core](https://nf-co.re): pipeline cộng đồng và quy ước phát triển pipeline trên GitHub.
- [GitHub Docs](https://docs.github.com): hướng dẫn chính thức, gồm SSH, Actions, secret scanning.
- [Learn Git Branching](https://learngitbranching.js.org): bài tập tương tác về nhánh.
