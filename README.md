# LINKED-LIST-BLOCK

#include <windows.h>
#include <string>
#include <ctime>
#include <sstream>
using namespace std;

struct Block {
    int index;
    string data;
    string timestamp;
    Block* next;

    Block(int idx, const string& d) : index(idx), data(d), next(nullptr) {
        time_t now = time(0);
        timestamp = ctime(&now);
        timestamp.pop_back();
    }
};

class BlockLink {
public:
    Block* head = nullptr;
    int blockCount = 0;

    void addBlock(const string& data) {
        int index = ++blockCount;
        Block* newBlock = new Block(index, data);
        if (!head) head = newBlock;
        else {
            Block* temp = head;
            while (temp->next) temp = temp->next;
            temp->next = newBlock;
        }
    }

    string displayBlocks() {
        ostringstream oss;
        if (!head) return "No blocks to display.";
        Block* temp = head;
        while (temp) {
            oss << "Block #" << temp->index << "\nData: " << temp->data << "\nTime: " << temp->timestamp << "\n\n";
            temp = temp->next;
        }
        return oss.str();
    }

    string searchBlock(const string& keyword) {
        ostringstream oss;
        Block* temp = head;
        while (temp) {
            if (temp->data.find(keyword) != string::npos)
                oss << "Found in Block #" << temp->index << ": " << temp->data << "\n";
            temp = temp->next;
        }
        return oss.str().empty() ? "No matching block found." : oss.str();
    }

    string deleteBlock(int idx) {
        Block *temp = head, *prev = nullptr;
        while (temp && temp->index != idx) {
            prev = temp;
            temp = temp->next;
        }
        if (!temp) return "Block not found.";
        if (!prev) head = temp->next;
        else prev->next = temp->next;
        delete temp;
        return "Block deleted.";
    }

    void clearAllBlocks() {
        while (head) {
            Block* temp = head;
            head = head->next;
            delete temp;
        }
        blockCount = 0;
    }
};

BlockLink blockchain;
HWND hOutput;

void SetOutputText(const string& text) {
    SetWindowTextA(hOutput, text.c_str());
}

string SimpleInputBox(HWND hwnd, const string& prompt) {
    char input[256] = "";

    HWND popup = CreateWindowEx(WS_EX_TOOLWINDOW, "STATIC", prompt.c_str(),
        WS_OVERLAPPED | WS_CAPTION | WS_SYSMENU,
        400, 300, 300, 150,
        hwnd, NULL, GetModuleHandle(NULL), NULL);

    HWND hEdit = CreateWindow("EDIT", "", WS_CHILD | WS_VISIBLE | WS_BORDER,
        20, 40, 240, 20, popup, NULL, GetModuleHandle(NULL), NULL);

    HWND hOk = CreateWindow("BUTTON", "OK", WS_CHILD | WS_VISIBLE | BS_DEFPUSHBUTTON,
        100, 80, 70, 25, popup, (HMENU)1, GetModuleHandle(NULL), NULL);

    ShowWindow(popup, SW_SHOW);
    UpdateWindow(popup);

    MSG msg;
    while (GetMessage(&msg, NULL, 0, 0)) {
        if (msg.message == WM_LBUTTONDOWN && msg.hwnd == hOk) {
            GetWindowTextA(hEdit, input, 256);
            DestroyWindow(popup);
            break;
        }
        TranslateMessage(&msg);
        DispatchMessage(&msg);
    }

    return string(input);
}

LRESULT CALLBACK WindowProc(HWND hwnd, UINT msg, WPARAM wParam, LPARAM lParam) {
    switch (msg) {
        case WM_COMMAND:
            switch (LOWORD(wParam)) {
                case 1: { // Add
                    string input = SimpleInputBox(hwnd, "Enter block data:");
                    if (!input.empty()) {
                        blockchain.addBlock(input);
                        SetOutputText("Block added.");
                    }
                    break;
                }
                case 2: // Display
                    SetOutputText(blockchain.displayBlocks());
                    break;
                case 3: { // Search
                    string keyword = SimpleInputBox(hwnd, "Enter keyword to search:");
                    if (!keyword.empty())
                        SetOutputText(blockchain.searchBlock(keyword));
                    break;
                }
                case 4: { // Delete
                    string input = SimpleInputBox(hwnd, "Enter block index to delete:");
                    try {
                        int id = stoi(input);
                        SetOutputText(blockchain.deleteBlock(id));
                    } catch (...) {
                        SetOutputText("Invalid index.");
                    }
                    break;
                }
                case 5: // Clear
                    blockchain.clearAllBlocks();
                    SetOutputText("All blocks cleared.");
                    break;
            }
            break;
        case WM_DESTROY:
            PostQuitMessage(0);
            return 0;
    }
    return DefWindowProc(hwnd, msg, wParam, lParam);
}

int WINAPI WinMain(HINSTANCE hInst, HINSTANCE, LPSTR, int nCmdShow) {
    const char CLASS_NAME[] = "BlockchainGUI";

    WNDCLASS wc = {};
    wc.lpfnWndProc = WindowProc;
    wc.hInstance = hInst;
    wc.lpszClassName = CLASS_NAME;
    wc.hbrBackground = (HBRUSH)(COLOR_WINDOW + 1);

    RegisterClass(&wc);

    HWND hwnd = CreateWindowEx(0, CLASS_NAME, "Blockchain Visual Simulator",
        WS_OVERLAPPEDWINDOW, CW_USEDEFAULT, CW_USEDEFAULT, 500, 400,
        NULL, NULL, hInst, NULL);

    hOutput = CreateWindow("EDIT", "", WS_CHILD | WS_VISIBLE | WS_BORDER | ES_MULTILINE | ES_AUTOVSCROLL | ES_READONLY,
        20, 100, 440, 200, hwnd, NULL, hInst, NULL);

    CreateWindow("BUTTON", "Add", WS_TABSTOP | WS_VISIBLE | WS_CHILD | BS_DEFPUSHBUTTON,
        20, 20, 70, 30, hwnd, (HMENU)1, hInst, NULL);
    CreateWindow("BUTTON", "Display", WS_TABSTOP | WS_VISIBLE | WS_CHILD | BS_DEFPUSHBUTTON,
        100, 20, 70, 30, hwnd, (HMENU)2, hInst, NULL);
    CreateWindow("BUTTON", "Search", WS_TABSTOP | WS_VISIBLE | WS_CHILD | BS_DEFPUSHBUTTON,
        180, 20, 70, 30, hwnd, (HMENU)3, hInst, NULL);
    CreateWindow("BUTTON", "Delete", WS_TABSTOP | WS_VISIBLE | WS_CHILD | BS_DEFPUSHBUTTON,
        260, 20, 70, 30, hwnd, (HMENU)4, hInst, NULL);
    CreateWindow("BUTTON", "Clear All", WS_TABSTOP | WS_VISIBLE | WS_CHILD | BS_DEFPUSHBUTTON,
        340, 20, 90, 30, hwnd, (HMENU)5, hInst, NULL);

    ShowWindow(hwnd, nCmdShow);
    UpdateWindow(hwnd);

    MSG msg = {};
    while (GetMessage(&msg, NULL, 0, 0)) {
        TranslateMessage(&msg);
        DispatchMessage(&msg);
    }
    return 0;
}
