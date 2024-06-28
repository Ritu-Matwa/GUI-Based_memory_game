# GUI-Based_memory_game
#include <QApplication>
#include <QWidget>
#include <QPushButton>
#include <QGridLayout>
#include <QMessageBox>
#include <QTimer>
#include <QLabel>
#include <QFont>
#include <QVBoxLayout>
#include <vector>
#include <utility>
#include <random>
#include <algorithm>

using namespace std;

class MemoryGame : public QWidget {
    Q_OBJECT

public:
    MemoryGame(QWidget *parent = nullptr);
    ~MemoryGame();

private slots:
    void handleButtonClick(int row, int col);
    void checkMatch();

private:
    void initializeGame();
    void shuffleSymbols();
    void createButtons();

    const int rows = 4;
    const int columns = 4;
    QGridLayout *gridLayout;
    QVBoxLayout *mainLayout;
    QLabel *movesLabel;
    vector<vector<QPushButton*>> buttons;
    vector<vector<QString>> symbols;
    vector<vector<bool>> revealed;
    pair<int, int> firstClick;
    pair<int, int> secondClick;
    bool firstSelected;
    QTimer *timer;
    int movesCount;
};

MemoryGame::MemoryGame(QWidget *parent)
    : QWidget(parent), firstSelected(false), timer(new QTimer(this)), movesCount(0) {
    gridLayout = new QGridLayout(this);
    mainLayout = new QVBoxLayout(this);
    movesLabel = new QLabel("Moves: 0", this);
    movesLabel->setAlignment(Qt::AlignCenter);
    QFont labelFont("Arial", 16, QFont::Bold);
    movesLabel->setFont(labelFont);

    initializeGame();
    createButtons();
    shuffleSymbols();

    mainLayout->addWidget(movesLabel);
    mainLayout->addLayout(gridLayout);
    setLayout(mainLayout);

    connect(timer, &QTimer::timeout, this, &MemoryGame::checkMatch);

    setWindowTitle("Memory Game");
    resize(400, 400);
}

MemoryGame::~MemoryGame() {}

void MemoryGame::initializeGame() {
    QStringList symbolList = {"A", "A", "B", "B", "C", "C", "D", "D", "E", "E", "F", "F", "G", "G", "H", "H"};
    symbols.resize(rows, vector<QString>(columns));
    revealed.resize(rows, vector<bool>(columns, false));

    int index = 0;
    for (int r = 0; r < rows; ++r) {
        for (int c = 0; c < columns; ++c) {
            symbols[r][c] = symbolList.at(index++);
        }
    }
}

void MemoryGame::createButtons() {
    buttons.resize(rows, vector<QPushButton*>(columns));
    QFont buttonFont("Arial", 18, QFont::Bold);
    for (int r = 0; r < rows; ++r) {
        for (int c = 0; c < columns; ++c) {
            QPushButton *btn = new QPushButton(this);
            btn->setFont(buttonFont);
            btn->setSizePolicy(QSizePolicy::Expanding, QSizePolicy::Expanding);
            btn->setStyleSheet("QPushButton { background-color: lightblue; } QPushButton:disabled { background-color: lightgreen; }");
            gridLayout->addWidget(btn, r, c);
            buttons[r][c] = btn;
            connect(btn, &QPushButton::clicked, [this, r, c]() { handleButtonClick(r, c); });
        }
    }
}

void MemoryGame::shuffleSymbols() {
    random_device rd;
    mt19937 g(rd());
    shuffle(symbols.begin(), symbols.end(), g);
    for (auto &row : symbols) {
        shuffle(row.begin(), row.end(), g);
    }
}

void MemoryGame::handleButtonClick(int row, int col) {
    if (revealed[row][col]) return;

    buttons[row][col]->setText(symbols[row][col]);
    buttons[row][col]->setEnabled(false);
    revealed[row][col] = true;

    if (!firstSelected) {
        firstClick = {row, col};
        firstSelected = true;
    } else {
        secondClick = {row, col};
        firstSelected = false;
        movesCount++;
        movesLabel->setText(QString("Moves: %1").arg(movesCount));
        timer->start(1000);
    }
}

void MemoryGame::checkMatch() {
    timer->stop();
    int r1 = firstClick.first, c1 = firstClick.second;
    int r2 = secondClick.first, c2 = secondClick.second;

    if (symbols[r1][c1] != symbols[r2][c2]) {
        buttons[r1][c1]->setText("");
        buttons[r1][c1]->setEnabled(true);
        buttons[r2][c2]->setText("");
        buttons[r2][c2]->setEnabled(true);
        revealed[r1][c1] = false;
        revealed[r2][c2] = false;
    }

    bool allMatched = true;
    for (const auto &row : revealed) {
        if (any_of(row.begin(), row.end(), [](bool v) { return !v; })) {
            allMatched = false;
            break;
        }
    }

    if (allMatched) {
        QMessageBox::information(this, "Memory Game", "Congratulations! You won in " + QString::number(movesCount) + " moves!");
        for (auto &row : buttons) {
            for (auto &btn : row) {
                btn->setEnabled(false);
            }
        }
    }
}

int main(int argc, char *argv[]) {
    QApplication app(argc, argv);
    MemoryGame game;
    game.show();
    return app.exec();
}

#include "main.moc"
