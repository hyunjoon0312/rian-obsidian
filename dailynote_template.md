<%*
// 파일 제목에서 날짜 추출
const fileDate = moment(tp.file.title, "YYYY-MM-DD");

// 참조할 날짜 배열 생성 (10일 전까지)
const referenceDates = Array.from({length: 10}, (_, i) => 
    fileDate.clone().subtract(i + 1, 'days').format("YYYY-MM-DD")
);

let incompleteTasks = [];
let referenceDate = "";
let referenceFile = null;
let filesFound = false;

// 파일 내용 가져오기 함수
async function getFileContent(date) {
    const file = tp.file.find_tfile(date);
    if (file) {
        referenceFile = file;
        filesFound = true;
        return await app.vault.read(file);
    }
    return null;
}

// 파일 찾기 및 내용 추출
for (const date of referenceDates) {
    const content = await getFileContent(date);
    if (content) {
        const todosSection = content.split("## 오늘 할일")[1];
        if (todosSection) {
            const todoContent = todosSection.split("## 일간 회고")[0].trim();
            incompleteTasks = todoContent.split("\n")
                .filter(line => line.trim().startsWith("- [ ]") && line.trim().length > 5)
                .map(line => line.trim());
            referenceDate = date;
            break;
        }
    }
}

// 결과 출력
if (referenceFile) {
    tR += `###### 전일 노트 : [[${referenceFile.basename}]]\n\n`;
} else if (!filesFound) {
    tR += "###### 전일 노트 : 최근 10일 이상 생성한 파일 없음.\n\n";
}

tR += "## 오늘 할일\n";
if (incompleteTasks.length > 0) {
    incompleteTasks.forEach(task => {
        tR += `${task} (${referenceDate})\n`;
    });
}

// 빈 체크박스 3개 추가
tR += "- [ ] \n".repeat(3);
%>



## 일간 회고
