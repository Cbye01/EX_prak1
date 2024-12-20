#include <windows.h>
#include <stdio.h>

int main() {
    HANDLE hReadPipe, hWritePipe;
    SECURITY_ATTRIBUTES saAttr;
    STARTUPINFO si;
    PROCESS_INFORMATION pi;
    DWORD dwWritten;
    const char *message = "Hello from parent process!\n";

    
    saAttr.nLength = sizeof(SECURITY_ATTRIBUTES);
    saAttr.bInheritHandle = TRUE;
    saAttr.lpSecurityDescriptor = NULL;

    
    if (!CreatePipe(&hReadPipe, &hWritePipe, &saAttr, 0)) {
        printf("CreatePipe failed (%d)\n", GetLastError());
        return 1;
    }

    
    ZeroMemory(&si, sizeof(STARTUPINFO));
    si.cb = sizeof(STARTUPINFO);
    si.hStdInput = hReadPipe;  
    si.hStdOutput = GetStdHandle(STD_OUTPUT_HANDLE);  
    si.dwFlags |= STARTF_USESTDHANDLES;

    
    ZeroMemory(&pi, sizeof(PROCESS_INFORMATION));
    if (!CreateProcess(
            NULL, 
            "cmd.exe",  // Запуск командного процесора
            NULL, NULL, TRUE, 0, NULL, NULL, &si, &pi)) {
        printf("CreateProcess failed (%d)\n", GetLastError());
        return 1;
    }

  
    CloseHandle(hReadPipe);
    CloseHandle(pi.hProcess);
    CloseHandle(pi.hThread);

    
    if (!WriteFile(hWritePipe, message, (DWORD)strlen(message), &dwWritten, NULL)) {
        printf("WriteFile failed (%d)\n", GetLastError());
        return 1;
    }

    
    CloseHandle(hWritePipe);

    
    WaitForSingleObject(pi.hProcess, INFINITE);

    
    CloseHandle(pi.hProcess);
    CloseHandle(pi.hThread);

    return 0;
}
