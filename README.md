#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <dirent.h>
#include <stdbool.h>
#include <unistd.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <time.h>

#define MAX_FILENAME_LENGTH 1000
#define MAX_COMMIT_MESSAGE_LENGTH 2000
#define MAX_LINE_LENGTH 1000
#define MAX_MESSAGE_LENGTH 1000
#define debug(x) printf("%s", x);

void add_n(int depth, int n);
void print_command(int argc, char *const argv[]);
int run_init(int argc, char *const argv[]);
int create_configs(char *username, char *email);
void run_add(char *file_to_add, char *cwd);
int add_to_staging(char *filepath);
void run_reset(int argc, char *data);
int remove_from_staging(char *filepath);
int run_commit(int argc, char *const argv[]);
int inc_last_commit_ID();
bool check_file_directory_exists(char *filepath);
int commit_staged_file(int commit_ID, char *filepath);
int track_file(char *filepath);
bool is_tracked(char *filepath);
int create_commit_file(int commit_ID, char *message, int counter_of_files_commiting);
int find_file_last_commit(char *filepath);
char *where_before();
char *standard_maker(char *name);
int config_global(char *mode, char *data);
void config_normal(char *mode, char *data);
void global_alias(char *old_command, char *new_command);
bool checker(char *name, char *address, char *ripo_address);
void reset_undo();
char *time_finder();
int *date_to_int(char *date);
void author_log(char *name);
void shortcut_use(char *shortcut_name);
void commits_log(int n);
int date_compare(int *input, int *file_date);
void time_log(char *input_date, char *mode);
void branch_show();
void branch_maker(char *branch_name);
void apply_checkout(char *number);
char *open_directory(char *name, char *number);
int how_many_commits();
void HEAD_status_to_zero();
void HEAD_status_to_one();
void branch_log(char *branch_name);
char *back_slash_n_remover(char *string);
char *null_space_remover_from_a_line(char *sentence);
void print_color(int color_code, const char *text);
void diff(char *first_file, char *second_file);
void grep_commit(int commit_id, char *word);
void grep_n(char *file_name, char *word);
void diff_line(char *first_file, char *second_file, int first_line, int second_line);
void print_color(int color_code, const char *text);
void todo_full_check();
void pre_commit();
int todo_check(char *filename);
char *getFileExtension(const char *filename);
int balance_braces(char *filename);
void balance_braces_full();
int character_limit(char *filename);
long getFileSize(const char *filename);
void file_size_check_full();

void print_command(int argc, char *const argv[])
{
    for (int i = 0; i < argc; i++)
    {
        fprintf(stdout, "%s ", argv[i]);
    }
    fprintf(stdout, "\n");
}

int run_init(int argc, char *const argv[])
{
    char cwd[1024];
    if (getcwd(cwd, sizeof(cwd)) == NULL)
        return 1;
    char tmp_cwd[1024];
    bool exists = false;
    struct dirent *entry;
    do
    {
        // find .neogit
        DIR *dir = opendir(".");
        if (dir == NULL)
        {
            perror("Error opening current directory");
            return 1;
        }
        while ((entry = readdir(dir)) != NULL)
        {
            if (entry->d_type == DT_DIR && strcmp(entry->d_name, ".neogit") == 0)
                exists = true;
        }
        closedir(dir);

        // update current working directory
        if (getcwd(tmp_cwd, sizeof(tmp_cwd)) == NULL)
            return 1;

        // change cwd to parent
        if (strcmp(tmp_cwd, "/") != 0)
        {
            if (chdir("..") != 0)
                return 1;
        }

    } while (strcmp(tmp_cwd, "/") != 0);

    // return to the initial cwd
    if (chdir(cwd) != 0)
        return 1;

    if (!exists)
    {
        if (mkdir(".neogit", 0755) != 0)
        {
            return 1;
        }
        char cwd[1024];
        getcwd(cwd, sizeof(cwd));
        char *username = (char *)malloc(100 * sizeof(char));
        char *useremail = (char *)malloc(100 * sizeof(char));
        chdir("/Users/arashghavami");
        FILE *file = fopen("username.txt", "r");
        fgets(username, 1000, file);
        if (*(username + strlen(username) - 1) == '\n')
        {
            *(username + strlen(username) - 1) = '\0';
        }
        fclose(file);
        file = fopen("useremail.txt", "r");
        fgets(useremail, 1000, file);
        if (*(useremail + strlen(useremail) - 1) == '\n')
        {
            *(useremail + strlen(useremail) - 1) = '\0';
        }
        fclose(file);
        chdir(cwd);
        return create_configs(username, useremail);
    }
    else
    {
        perror("neogit repository has already initialized");
    }
    return 0;
}

int create_configs(char *username, char *email)
{
    FILE *file = fopen(".neogit/config", "w");
    if (file == NULL)
        return 1;

    fprintf(file, "username: %s\n", username);
    fprintf(file, "email: %s\n", email);
    fprintf(file, "last_commit_ID: %d\n", 0);
    fprintf(file, "current_commit_ID: %d\n", 0);
    fprintf(file, "branch: %s", "master");
    fclose(file);
    // create  folder
    if (mkdir(".neogit/commits", 0755) != 0)
        return 1;
    // create files folder
    if (mkdir(".neogit/files", 0755) != 0)
        return 1;
    if (mkdir(".neogit/staging_area", 0755) != 0)
        return 1;
    if (mkdir(".neogit/branches", 0755) != 0)
        return 1;
    if (mkdir(".neogit/branches/master", 0755) != 0)
        return 1;
    if (mkdir(".neogit/tags", 0755) != 0)
        return 1;

    file = fopen(".neogit/HEAD_status", "w");
    fprintf(file, "0");
    fclose(file);
    file = fopen(".neogit/staging", "w");
    fclose(file);
    file = fopen(".neogit/tracks", "w");
    fclose(file);
    file = fopen(".neogit/precommit", "w");
    fclose(file);
    file = fopen(".neogit/precommit_result", "w");
    fclose(file);
    //
    file = fopen(".neogit/alias", "w");
    FILE *file1 = fopen("/Users/arashghavami/alias.txt", "r");
    char *temp = (char *)malloc(100 * sizeof(char));
    while (fgets(temp, 1000, file1) != NULL)
    {
        fprintf(file, "%s", temp);
    }
    return 0;
}

void run_add(char *file_to_add, char *cwd)
{
    char *address = (char *)malloc(200 * sizeof(char));
    address = where_before();
    strcat(cwd, "/");
    strcat(cwd, file_to_add);
    char *send = (char *)malloc(200 * sizeof(char));
    if (access(cwd, F_OK) != 0)
    {
        printf("%s does not exist\n", file_to_add);
        return;
    }
    int counter = 0;
    for (int i = strlen(address) + 1; i < strlen(cwd); i++)
    {
        *(send + counter) = cwd[i];
        counter++;
    }
    add_to_staging(send);
    // copy and paste the file or the directory to the repository:
    char *command = (char *)malloc(200 * sizeof(char));
    strcpy(command, "cp -r ");
    strcat(command, cwd);
    strcat(command, " ");
    char *address2 = where_before();
    strcat(address2, "/.neogit/staging_area/");
    for (int i = 0; i < strlen(send); i++)
    {
        if (*(send + i) == '/')
        {
            *(send + i) = '-';
        }
    }
    strcat(address2, send);
    strcat(command, address2);
    system(command);
}

int add_to_staging(char *filepath)
{
    char *address = (char *)malloc(200 * sizeof(char));
    address = where_before();
    strcat(address, "/.neogit/staging");
    FILE *file = fopen(address, "r");
    if (file == NULL)
        return 1;
    char line[MAX_LINE_LENGTH];
    while (fgets(line, sizeof(line), file) != NULL)
    {
        int length = strlen(line);
        if (length > 0 && line[length - 1] == '\n')
        {
            line[length - 1] = '\0';
        }

        if (strcmp(filepath, line) == 0)
            return 0;
    }
    fclose(file);
    file = fopen(address, "a");
    if (file == NULL)
        return 1;

    fprintf(file, "%s\n", filepath);
    fclose(file);
    return 0;
}

void run_reset(int argc, char *data)
{
    char *file_path = (char *)malloc(100 * sizeof(char));
    file_path = standard_maker(data);
    if (argc < 3)
    {
        perror("please specify a file");
        return;
    }
    printf("file path = %s\n", file_path);
    remove_from_staging(file_path);
}

int remove_from_staging(char *filepath)
{
    FILE *file = fopen(".neogit/staging", "r");
    if (file == NULL)
        return 1;

    FILE *tmp_file = fopen(".neogit/tmp_staging", "w");
    if (tmp_file == NULL)
        return 1;

    bool exist = false;
    char line[MAX_LINE_LENGTH];
    while (fgets(line, sizeof(line), file) != NULL)
    {
        int length = strlen(line);

        // remove '\n'
        if (length > 0 && line[length - 1] == '\n')
        {
            line[length - 1] = '\0';
        }

        if (strcmp(filepath, line) != 0)
        {
            // fputs(line, tmp_file);
            fprintf(tmp_file, "%s", line);
            fprintf(tmp_file, "%c", '\n');
        }
        else
        {
            exist = true;
        }
    }
    if (exist)
    {
        FILE *file2 = fopen(".neogit/reset", "a");
        char *temp = (char *)malloc(100 * sizeof(char));
        fprintf(file2, "%s\n", filepath);
        fclose(file2);
    }
    fclose(file);
    fclose(tmp_file);

    remove(".neogit/staging");
    rename(".neogit/tmp_staging", ".neogit/staging");

    // remove from directory:
    char *address = where_before();
    strcat(address, "/.neogit/staging_area");
    chdir(address);
    for (int i = 0; i < strlen(filepath); i++)
    {
        if (*(filepath + i) == '/')
        {
            *(filepath + i) = '-';
        }
    }
    remove(filepath);

    return 0;
}

int run_commit(int argc, char *const argv[])
{
    char *address3 = where_before();
    chdir(address3);
    chdir(".neogit");
    FILE *file8 = fopen("HEAD_status", "r");
    fscanf(file8, "%s", address3);
    if (strcmp(address3, "0") != 0)
    {
        printf("you should checkout to the HEAD of your project to ba able to commit");
        return 1;
    }
    char *address2 = where_before();
    chdir(address2);
    chdir(".neogit");
    if (argc < 4)
    {
        perror("please use the correct format");
        return 1;
    }
    char message[MAX_MESSAGE_LENGTH];
    strcpy(message, argv[3]);
    if (strlen(message) > 72)
    {
        printf("your commit message has more than 72 charecters");
        return 1;
    }
    char *address = where_before();
    strcat(address, "/.neogit/staging");
    FILE *file = fopen(address, "r");
    if (file == NULL)

        return 1;
    if (fgetc(file) == EOF)
    {
        printf("your staging area is empty");
        return 1;
    }
    fseek(file, 0, SEEK_SET);
    int commit_ID = inc_last_commit_ID();
    if (commit_ID == -1)
        return 1;
    char line[MAX_LINE_LENGTH];
    int counter_of_files_commiting = 0;
    while (fgets(line, sizeof(line), file) != NULL)
    {
        counter_of_files_commiting++;
        int length = strlen(line);
        if (length > 0 && line[length - 1] == '\n')
        {
            line[length - 1] = '\0';
        }

        if (!check_file_directory_exists(line))
        {
            address = where_before();
            chdir("files");
            if (mkdir(line, 0755) != 0)
                return 1;
        }
        printf("commit %s\n", line);
        commit_staged_file(commit_ID, line);
        for (int i = 0; i < strlen(line); i++)
        {
            if (*(line + i) == '-')
            {
                *(line + i) = '/';
            }
        }
        track_file(line);
    }
    char cwd3[1000];
    getcwd(cwd3, sizeof(cwd3));
    char *add = where_before();
    strcat(add, "/.neogit");
    chdir(add);
    char *command = (char *)malloc(100 * sizeof(char));
    strcpy(command, "rm -r staging_area");
    system(command);
    strcpy(command, "mkdir staging_area");
    system(command);
    chdir(cwd3);
    fclose(file);

    file = fopen(".neogit/staging", "w");
    if (file == NULL)
        return 1;
    fclose(file);
    // printf("fuck");
    create_commit_file(commit_ID, message, counter_of_files_commiting);
    fprintf(stdout, "commit successfully with commit ID %d\n", commit_ID);
    printf("commit message: %s\n", argv[3]);
    char *time = time_finder();
    printf("time: %s", time);

    return 0;
}
// returns new commit_ID
int inc_last_commit_ID()
{
    char *address = where_before();
    strcat(address, "/.neogit/config");
    FILE *file = fopen(address, "r");
    if (file == NULL)
        return -1;
    address = where_before();
    strcat(address, "/.neogit/tmp_config");
    FILE *tmp_file = fopen(address, "w");
    if (tmp_file == NULL)
        return -1;

    int last_commit_ID;
    char line[MAX_LINE_LENGTH];
    while (fgets(line, sizeof(line), file) != NULL)
    {
        if (strncmp(line, "last_commit_ID", 14) == 0)
        {
            sscanf(line, "last_commit_ID: %d\n", &last_commit_ID);
            last_commit_ID++;
            fprintf(tmp_file, "last_commit_ID: %d\n", last_commit_ID);
        }
        else
            fprintf(tmp_file, "%s", line);
    }
    fclose(file);
    fclose(tmp_file);

    remove(".neogit/config");
    rename(".neogit/tmp_config", ".neogit/config");
    return last_commit_ID;
}

bool check_file_directory_exists(char *filepath)
{
    char *nametofind = (char *)malloc(100 * sizeof(char));
    for (int i = 0; i < strlen(filepath); i++)
    {
        *(nametofind + i) = *(filepath + i);
    }
    for (int i = 0; i < strlen(nametofind); i++)
    {
        if (*(nametofind + i) == '/')
        {
            *(nametofind + i) = '-';
        }
    }
    DIR *dir = opendir("files");
    struct dirent *entry;
    if (dir == NULL)
    {
        // perror("Error opening current directory");
        return 1;
    }
    while ((entry = readdir(dir)) != NULL)
    {
        if (entry->d_type == DT_DIR && strcmp(entry->d_name, nametofind) == 0)
            return true;
    }
    closedir(dir);

    return false;
}

int commit_staged_file(int commit_ID, char *filepath)
{
    for (int i = 0; i < strlen(filepath); i++)
    {
        if (*(filepath + i) == '/')
        {
            *(filepath + i) = '-';
        }
    }
    char *beginning = (char *)malloc(100 * sizeof(char));
    char *destination = (char *)malloc(100 * sizeof(char));
    char *temp = where_before();
    strcpy(beginning, temp);
    strcat(beginning, "/.neogit/staging_area/");
    strcat(beginning, filepath);
    destination = where_before();
    strcat(destination, "/.neogit/files/");
    strcat(destination, filepath);
    char *command = (char *)malloc(1000 * sizeof(char));
    strcpy(command, "cp ");
    strcat(command, beginning);
    strcat(command, " ");
    strcat(command, destination);
    mkdir(destination, 0755);
    system(command);
    char cwd[500];
    getcwd(cwd, sizeof(cwd));
    chdir(destination);
    char newname[10];
    sprintf(newname, "%d", commit_ID);
    strcat(destination, "/");
    strcat(destination, filepath);
    rename(destination, newname);
    chdir(cwd);
    return 0;
}

int track_file(char *filepath)
{
    if (is_tracked(filepath))
        return 0;

    FILE *file = fopen(".neogit/tracks", "a");
    if (file == NULL)
        return 1;
    fprintf(file, "%s\n", filepath);
    return 0;
}

bool is_tracked(char *filepath)
{
    FILE *file = fopen(".neogit/tracks", "r");
    if (file == NULL)
        return false;
    char line[MAX_LINE_LENGTH];
    while (fgets(line, sizeof(line), file) != NULL)
    {
        int length = strlen(line);

        // remove '\n'
        if (length > 0 && line[length - 1] == '\n')
        {
            line[length - 1] = '\0';
        }

        if (strcmp(line, filepath) == 0)
            return true;
    }
    fclose(file);

    return false;
}

int create_commit_file(int commit_ID, char *message, int counter_of_files_commiting)
{

    char commit_filepath[MAX_FILENAME_LENGTH];
    char *temp = where_before();
    strcpy(commit_filepath, temp);
    strcat(commit_filepath, "/.neogit/commits/");
    char tmp[10];
    sprintf(tmp, "%d", commit_ID);
    strcat(commit_filepath, tmp);
    FILE *file = fopen(commit_filepath, "w");
    if (file == NULL)
    {
        printf("error opening the commit data file");
        return 1;
    }

    fprintf(file, "message: %s\n", message);
    fprintf(file, "files:\n");
    DIR *dir = opendir(".");
    struct dirent *entry;
    if (dir == NULL)
    {
        perror("Error opening current directory");
        return 1;
    }
    while ((entry = readdir(dir)) != NULL)
    {
        if (entry->d_type == DT_REG && is_tracked(entry->d_name))
        {
            int file_last_commit_ID = find_file_last_commit(entry->d_name);
            fprintf(file, "%s %d\n", entry->d_name, file_last_commit_ID);
        }
    }
    closedir(dir);
    char *time = time_finder();
    fprintf(file, "time: %s\n", time);
    char *address = (char *)malloc(100 * sizeof(char));
    char *temp2 = where_before();
    strcpy(address, temp2);
    strcat(address, "/.neogit/config");
    FILE *file2 = fopen(address, "r");
    fprintf(file, "author: ");
    char *data_to_transfer = (char *)malloc(1000 * sizeof(char));
    fgets(data_to_transfer, 1000, file2);
    data_to_transfer = strtok(data_to_transfer, " ");
    data_to_transfer = strtok(NULL, "\n");
    fprintf(file, "%s", data_to_transfer);
    fprintf(file, "\n");
    fprintf(file, "email: ");
    fgets(data_to_transfer, 1000, file2);
    data_to_transfer = strtok(data_to_transfer, " ");
    data_to_transfer = strtok(NULL, "\n");
    fprintf(file, "%s", data_to_transfer);
    fprintf(file, "\nnumber of files in this commit: ");
    fprintf(file, "%d\n", counter_of_files_commiting);
    char *tp = (char *)malloc(200 * sizeof(char));
    tp = where_before();
    strcat(tp, "/.neogit/config");
    FILE *my_file = fopen(tp, "r");
    for (int i = 0; i < 5; i++)
    {
        fgets(tp, 1000, my_file);
    }
    if (*(tp + strlen(tp) - 1) == '\n')
    {
        *(tp + strlen(tp) - 1) = '\0';
    }
    fprintf(file, "%s", tp);
    tp = strtok(tp, " ");
    tp = strtok(NULL, " ");
    fclose(file);
    char *branch_address = (char *)malloc(100 * sizeof(char));
    branch_address = where_before();
    strcat(branch_address, "/.neogit/branches/");
    strcat(branch_address, tp);
    char *command = (char *)malloc(200 * sizeof(char));
    strcpy(command, "cp ");
    strcat(command, commit_filepath);
    strcat(command, " ");
    strcat(command, branch_address);
    system(command);
    printf("command = %s\n", command);
    return 0;
}

int find_file_last_commit(char *filepath)
{
    char filepath_dir[MAX_FILENAME_LENGTH];
    strcpy(filepath_dir, ".neogit/files/");
    strcat(filepath_dir, filepath);

    int max = -1;

    DIR *dir = opendir(filepath_dir);
    struct dirent *entry;
    if (dir == NULL)
        return 1;

    while ((entry = readdir(dir)) != NULL)
    {
        if (entry->d_type == DT_REG)
        {
            int tmp = atoi(entry->d_name);
            max = max > tmp ? max : tmp;
        }
    }
    closedir(dir);

    return max;
}

int config_global(char *mode, char *data)
{
    chdir("/Users/arashghavami");
    DIR *dir = opendir(".");
    if (dir == NULL)
    {
        perror("Error opening home directory");
        return 1;
    }
    char *name = (char *)malloc(100 * sizeof(char));
    strcpy(name, mode);
    strcat(name, ".txt");
    FILE *file;
    file = fopen(name, "a");
    fprintf(file, "%s ", data);
    fclose(file);

    return 1;
}

char *where_before()
{
    char cwd[1024];
    if (getcwd(cwd, sizeof(cwd)) == NULL)
    {
        return NULL;
    }
    char result[1024];
    while (strcmp(cwd, "/") != 0)
    {
        struct dirent *entry;
        DIR *dir = opendir(".");
        while ((entry = readdir(dir)) != NULL)
        {
            if (entry->d_type == DT_DIR && strcmp(entry->d_name, ".neogit") == 0)
            {
                char tmp[1000];
                getcwd(tmp, sizeof(tmp));
                char *hi = (char *)malloc(1000 * sizeof(char));
                for (int i = 0; i < strlen(tmp); i++)
                {
                    *(hi + i) = tmp[i];
                }
                return hi;
            }
        }
        chdir("..");
        getcwd(cwd, sizeof(cwd));
    }
    return NULL;
}

void config_normal(char *mode, char *data)
{
    char *address = (char *)malloc(100 * sizeof(char));
    address = where_before();
    chdir(address);
    chdir(".neogit");
    FILE *file = fopen("config", "r");
    FILE *file2 = fopen("tmp", "w");
    char *temp = (char *)malloc(100 * sizeof(char));
    char x;
    if (strcmp(mode, "username") == 0)
    {
        fprintf(file2, "username: %s\n", data);
        fgets(temp, 1000, file);
        int counter = 0;
        while (fgets(temp, 1000, file) != NULL)
        {
            counter++;
            fprintf(file2, "%s", temp);
        }
    }
    else
    {
        fgets(temp, 1000, file);
        fprintf(file2, "%s", temp);
        fprintf(file2, "email: %s\n", data);
        int counter = 0;
        while (fgets(temp, 1000, file) != NULL)
        {
            if (counter == 1)
                fprintf(file2, "%s", temp);
            counter = 1;
        }
    }
    fclose(file);
    fclose(file2);
    free(temp);
    remove("config");
    rename("tmp", "config");
}

void global_alias(char *old_command, char *new_command)
{
    chdir("/Users/arashghavami");
    DIR *dir = opendir(".");
    bool exists = false;
    struct dirent *entry;
    while ((entry = readdir(dir)) != NULL)
    {
        if (strcmp(entry->d_name, "alias.txt") == 0)
        {
            exists = true;
        }
    }
    if (exists)
    {
        FILE *file1 = fopen("alias.txt", "r");
        FILE *file2 = fopen("t.txt", "w");
        char *temp = (char *)malloc(100 * sizeof(char));
        while (fgets(temp, 1000, file1) != NULL)
        {
            fprintf(file2, "%s", temp);
        }
        fclose(file1);
        fprintf(file2, "%s: %s\n", old_command, new_command);
        fclose(file2);
        free(temp);
        remove("alias.txt");
        rename("t.txt", "alias.txt");
    }
    else
    {
        FILE *file = fopen("alias.txt", "w");
        fprintf(file, "%s: %s\n", old_command, new_command);
        fclose(file);
    }
}

bool checker(char *name, char *address, char *ripo_address)
{
    // address = current working directory
    // ripo_address = where_before()
    // name = file name
    int x = strlen(ripo_address);
    strcat(ripo_address, "/.neogit/staging");
    FILE *file = fopen(ripo_address, "r");
    char *temp = (char *)malloc(100 * sizeof(char));
    char *standard = (char *)malloc(100 * sizeof(char));
    strcat(address, "/");
    strcat(address, name);
    int counter = 0;
    for (int i = x + 1; i < strlen(address); i++)
    {
        *(standard + counter) = *(address + i);
        counter++;
    }
    while (fgets(temp, 1000, file) != NULL)
    {
        if (*(temp + strlen(temp) - 1) == '\n')
        {
            *(temp + strlen(temp) - 1) = '\0';
        }
        if (strcmp(temp, standard) == 0)
        {
            return true;
        }
    }
    return false;
}

bool checker2(char *data)
{
    char *temp = (char *)malloc(100 * sizeof(char));
    char *ripo_address = where_before();
    strcat(ripo_address, "/.neogit/staging");
    FILE *file = fopen(ripo_address, "r");
    if (file == NULL)
    {
        perror("Error opening file");
        // Handle the error or return false
        return false;
    }
    while (fgets(temp, 100, file) != NULL)
    {
        if (*(temp + strlen(temp) - 1) == '\n')
        {
            *(temp + strlen(temp) - 1) = '\0';
        }
        if (strcmp(temp, data) == 0)
        {
            free(temp);
            return true;
        }
    }
    free(temp);
    return false;
}

char *standard_maker(char *name)
{
    char cwd[1000];
    getcwd(cwd, sizeof(cwd));
    char *address = where_before();
    char *result = (char *)malloc(100 * sizeof(char));
    int counter = 0;
    for (int i = strlen(address) + 1; i < strlen(cwd); i++)
    {
        *(result + counter) = cwd[i];
        counter++;
    }
    strcat(result, "/");
    strcat(result, name);
    char *result2 = (char *)malloc(1000 * sizeof(char));
    if (*(result + 0) == '/')
    {
        strcpy(result2, result + 1);
    }
    else
    {
        strcpy(result2, result);
    }
    return result2;
}

void reset_undo()
{
    char backslash_n_reader;
    char *address = where_before();
    strcat(address, "/.neogit/staging");
    FILE *file = fopen(address, "r");
    int counter = 0;
    while ((backslash_n_reader = fgetc(file)) != EOF)
    {
        if (backslash_n_reader == '\n')
        {
            counter++;
        }
    }
    fseek(file, 0, SEEK_SET);
    char *last_sentence = (char *)malloc(1000 * sizeof(char));
    address = where_before();
    strcat(address, "/.neogit/tmp");
    FILE *file2 = fopen(address, "w");
    for (int i = 0; i < counter - 1; i++)
    {
        fgets(last_sentence, 1000, file);
        fprintf(file2, "%s", last_sentence);
    }
    fgets(last_sentence, 1000, file);
    address = where_before();
    strcat(address, "/.neogit/reset");
    FILE *file3 = fopen(address, "a");
    fprintf(file3, "%s", last_sentence);
    fclose(file3);
    fclose(file2);
    fclose(file);
    address = where_before();
    strcat(address, "/.neogit/staging");
    remove(address);
    address = where_before();
    strcat(address, "/.neogit");
    chdir(address);
    rename("tmp", "staging");
    address = where_before();
    strcat(address, "/.neogit/staging_area");
    chdir(address);
    if (*(last_sentence + strlen(last_sentence) - 1) == '\n')
    {
        *(last_sentence + strlen(last_sentence) - 1) = '\0';
    }
    for (int i = 0; i < strlen(last_sentence); i++)
    {
        if (*(last_sentence + i) == '/')
        {
            *(last_sentence + i) = '-';
        }
    }
    remove(last_sentence);
}

void iterate_and_write_to_file(const char *directory, int depth, FILE *file)
{
    if (depth < 0)
    {
        return;
    }

    DIR *dir;
    struct dirent *entry;

    dir = opendir(directory);
    if (dir == NULL)
    {
        perror("Error opening directory");
        exit(EXIT_FAILURE);
    }

    while ((entry = readdir(dir)) != NULL)
    {
        if (strcmp(entry->d_name, ".") == 0 || strcmp(entry->d_name, "..") == 0)
        {
            continue;
        }

        // Construct the full path of the current entry
        char path[1000];
        snprintf(path, sizeof(path), "%s/%s", directory, entry->d_name);

        // Check if it's a directory
        struct stat statbuf;
        if (stat(path, &statbuf) == 0 && S_ISDIR(statbuf.st_mode))
        {
            fprintf(file, "%s/ ", path);
            char *address = where_before();
            int counter = 0;
            char *result = (char *)malloc(100 * sizeof(char));
            for (int i = strlen(address) + 1; i < strlen(path); i++)
            {
                *(result + counter) = path[i];
                counter++;
            }
            checker2(result);
            iterate_and_write_to_file(path, depth - 1, file);
        }
        else
        {
            char *address = where_before();
            int counter = 0;
            char *result = (char *)malloc(100 * sizeof(char));
            for (int i = strlen(address) + 1; i < strlen(path); i++)
            {
                *(result + counter) = path[i];
                counter++;
            }
            fprintf(file, "%s ", path);
            if (checker2(result))
            {
                fprintf(file, "---> staged");
            }
            else
            {
                fprintf(file, "---> unstaged");
            }
            fprintf(file, "%c", '\n');
        }
    }

    closedir(dir);
}

char *time_finder()
{
    time_t currentTime;
    time(&currentTime);
    char *timeString = ctime(&currentTime);
    if (*(timeString + strlen(timeString) - 1) == '\n')
    {
        *(timeString + strlen(timeString) - 1) = '\0';
    }
    return timeString;
}

void commits_log(int n)
{
    char *address = where_before();
    char *commits = (char *)malloc(1000 * sizeof(char));
    strcpy(commits, address);
    strcat(commits, "/.neogit/commits");
    chdir(commits);
    char *temp = (char *)malloc(100 * sizeof(char));
    struct dirent *entry;
    DIR *dir = opendir(".");
    printf("your commits log:\n");
    printf(".......................................\n");
    int counter = 0;
    while ((entry = readdir(dir)) != NULL)
    {

        if (strcmp(entry->d_name, ".") == 0 || strcmp(entry->d_name, "..") == 0 || strcmp(entry->d_name, ".DS_Store") == 0)
        {
            continue;
        }
        else
        {
            counter++;
        }
    }
    int x = counter - n + 1;
    if (n == -1)
    {
        x = 1;
    }
    for (int i = counter; i >= x; i--)
    {
        char *name = (char *)malloc(10 * sizeof(char));
        sprintf(name, "%d", i);
        printf("commit hash: %s\n", name);
        FILE *file = fopen(name, "r");
        fgets(temp, 100, file);
        printf("%s", temp);
        while (fscanf(file, "%s", temp) != EOF)
        {
            if (strcmp(temp, "time:") == 0)
            {
                break;
            }
        }
        fgets(temp, 1000, file);
        for (int i = 0; i < strlen(temp); i++)
        {
            if (*(temp + i) == '\n')
            {
                *(temp + i) = '\0';
            }
        }
        printf("time:%s\n", temp);
        fgets(temp, 1000, file);
        printf("%s", temp);
        fgets(temp, 1000, file);
        printf("%s", temp);
        fgets(temp, 1000, file);
        printf("%s", temp);
        fgets(temp, 1000, file);
        printf("%s\n", temp);
        printf(".......................................\n");
    }
}

void shortcut_use(char *shortcut_name)
{
    char *address = (char *)malloc(200 * sizeof(char));
    char *temp = where_before();
    strcpy(address, temp);
    strcat(address, "/.neogit/shortcuts");
    FILE *file = fopen(address, "r");
    strcat(shortcut_name, ":");
    int flag = 0;
    while (fscanf(file, "%s", temp) != EOF)
    {
        if (strcmp(temp, shortcut_name) == 0)
        {
            fgets(temp, 1000, file);
            if (*(temp + strlen(temp) - 1) == '\n')
            {
                *(temp + strlen(temp) - 1) = '\0';
            }
            flag = 1;
            char **argv2 = (char **)malloc(5 * sizeof(char *));
            for (int i = 0; i < 5; i++)
            {
                *(argv2 + i) = (char *)malloc(100 * sizeof(char));
            }
            strcpy(*(argv2 + 0), "neogit");
            strcpy(*(argv2 + 1), "commit");
            strcpy(*(argv2 + 2), "-m");
            strcpy(*(argv2 + 3), temp);
            run_commit(4, argv2);
            return;
        }
        else
        {
            fgets(temp, 1000, file);
        }
    }
    if (flag == 0)
    {
        printf("there is no such this shortcut\n");
        return;
    }
}

void author_log(char *name)
{
    char *address = (char *)malloc(100 * sizeof(char));
    address = where_before();
    strcat(address, "/.neogit/commits");
    chdir(address);
    struct dirent *entry;
    int counter = 0;
    DIR *dir = opendir(".");
    int flag = 0;
    while ((entry = readdir(dir)) != NULL)
    {
        if (strcmp(entry->d_name, ".") != 0 && strcmp(entry->d_name, "..") != 0 && strcmp(entry->d_name, ".DS_Store") != 0)
        {
            counter++;
        }
    }
    for (int i = counter; i >= 1; i--)
    {
        char tmp[5];
        sprintf(tmp, "%d", i);
        FILE *file = fopen(tmp, "r");
        char *temp = (char *)malloc(100 * sizeof(char));
        while (fscanf(file, "%s", temp) != EOF)
        {
            if (strcmp(temp, "author:") == 0)
            {
                break;
            }
            fgets(temp, 1000, file);
        }
        fgets(temp, 1000, file);
        for (int i = 0; i < strlen(temp); i++)
        {
            if (*(temp + i) == '\n')
            {
                *(temp + i) = '\0';
            }
        }
        if (*(temp + strlen(temp) - 1) == ' ')
        {
            *(temp + strlen(temp) - 1) = '\0';
        }
        int counter2 = 0;
        while (*(temp + counter2) != ' ')
        {
            counter2++;
        }
        temp = temp + counter2 + 1;
        if (strcmp(temp, name) == 0)
        {
            flag = 1;
            printf("commit hash: %s\n", tmp);
            fseek(file, 0, SEEK_SET);
            fgets(temp, 1000, file);
            if (*(temp + strlen(temp) - 1) == '\n')
            {
                *(temp + strlen(temp) - 1) = '\0';
            }
            printf("%s\n", temp);
            while (fscanf(file, "%s", temp) != EOF)
            {
                if (strcmp(temp, "time:") == 0)
                {
                    break;
                }
            }
            fgets(temp, 1000, file);
            for (int i = 0; i < strlen(temp); i++)
            {
                if (*(temp + i) == '\n')
                {
                    *(temp + i) = '\0';
                }
            }
            printf("time:%s\n", temp);
            fgets(temp, 1000, file);
            printf("%s", temp);
            fgets(temp, 1000, file);
            printf("%s", temp);
            fgets(temp, 1000, file);
            printf("%s", temp);
            fgets(temp, 1000, file);
            printf("%s\n", temp);
            printf(".......................................\n");
        }
        fclose(file);
    }
    if (flag == 0)
    {
        printf("this author has not comitted anything");
        return;
    }
}

int *date_to_int(char *date)
{
    int year, months, day;
    date = strtok(date, " ");
    if (strcmp(date, "Jan") == 0)
    {
        months = 1;
    }
    else if (strcmp(date, "Feb") == 0)
    {
        months = 2;
    }
    else if (strcmp(date, "Mar") == 0)
    {
        months = 3;
    }
    else if (strcmp(date, "Apr") == 0)
    {
        months = 4;
    }
    else if (strcmp(date, "May") == 0)
    {
        months = 5;
    }
    else if (strcmp(date, "Jun") == 0)
    {
        months = 6;
    }
    else if (strcmp(date, "Jul") == 0)
    {
        months = 7;
    }
    else if (strcmp(date, "Aug") == 0)
    {
        months = 8;
    }
    else if (strcmp(date, "Sep") == 0)
    {
        months = 9;
    }
    else if (strcmp(date, "Oct") == 0)
    {
        months = 10;
    }
    else if (strcmp(date, "Nov") == 0)
    {
        months = 11;
    }
    else if (strcmp(date, "Dec") == 0)
    {
        months = 12;
    }
    date = strtok(NULL, " ");
    sscanf(date, "%d", &day);
    date = strtok(NULL, " ");
    date = strtok(NULL, " ");
    sscanf(date, "%d", &year);
    int *dd = (int *)malloc(5 * sizeof(int));
    *(dd + 0) = day;
    *(dd + 1) = months;
    *(dd + 2) = year;
    return dd;
}

int date_compare(int *input, int *file_date)
{
    // input = ooni ke khod yaroo vared karde
    // file date = ooni ke too commit ha hast

    if (*(input + 2) > *(file_date + 2))
    {
        return 1;
    }
    else if (*(input + 2) < *(file_date + 2))
    {
        return -1;
    }

    if (*(input + 1) > *(file_date + 1))
    {
        return 1;
    }
    else if (*(input + 1) < *(file_date + 1))
    {
        return -1;
    }
    if (*(input + 0) > *(file_date + 0))
    {
        return 1;
    }
    else if (*(input + 0) < *(file_date + 0))
    {
        return -1;
    }
    return 0;
}

void time_log(char *input_date, char *mode)
{
    char *address = (char *)malloc(100 * sizeof(char));
    address = where_before();
    strcat(address, "/.neogit/commits");
    chdir(address);
    struct dirent *entry;
    int counter = 0;
    DIR *dir = opendir(".");
    int flag = 0;
    while ((entry = readdir(dir)) != NULL)
    {
        if (strcmp(entry->d_name, ".") != 0 && strcmp(entry->d_name, "..") != 0 && strcmp(entry->d_name, ".DS_Store") != 0)
        {
            counter++;
        }
    }
    closedir(dir);
    int *input = (int *)malloc(10 * sizeof(int));
    int day, months, year;
    sscanf(input_date, "%d", &day);
    input_date = strtok(input_date, " ");
    input_date = strtok(NULL, " ");
    if (strcmp(input_date, "Jan") == 0)
    {
        months = 1;
    }
    else if (strcmp(input_date, "Feb") == 0)
    {
        months = 2;
    }
    else if (strcmp(input_date, "Mar") == 0)
    {
        months = 3;
    }
    else if (strcmp(input_date, "Apr") == 0)
    {
        months = 4;
    }
    else if (strcmp(input_date, "May") == 0)
    {
        months = 5;
    }
    else if (strcmp(input_date, "Jun") == 0)
    {
        months = 6;
    }
    else if (strcmp(input_date, "Jul") == 0)
    {
        months = 7;
    }
    else if (strcmp(input_date, "Aug") == 0)
    {
        months = 8;
    }
    else if (strcmp(input_date, "Sep") == 0)
    {
        months = 9;
    }
    else if (strcmp(input_date, "Oct") == 0)
    {
        months = 10;
    }
    else if (strcmp(input_date, "Nov") == 0)
    {
        months = 11;
    }
    else if (strcmp(input_date, "Dec") == 0)
    {
        months = 12;
    }
    input_date = strtok(NULL, " ");
    sscanf(input_date, "%d", &year);
    *(input + 0) = day;
    *(input + 1) = months;
    *(input + 2) = year;

    for (int i = counter; i >= 1; i--)
    {
        char tmp[5];
        sprintf(tmp, "%d", i);
        FILE *file = fopen(tmp, "r");
        char *temp = (char *)malloc(100 * sizeof(char));
        while (fscanf(file, "%s", temp) != EOF)
        {
            if (*(temp + strlen(temp) - 1) == '\n')
            {
                *(temp + strlen(temp) - 1) = '\0';
            }
            if (strcmp(temp, "time:") == 0)
            {
                break;
            }
        }
        fgets(temp, 100, file);
        temp = temp + 4;
        int *file_date = date_to_int(temp);
        int status = date_compare(input, file_date);
        fseek(file, 0, SEEK_SET);
        if (strcmp(mode, "-since") == 0)
        {
            if (status == -1 || status == 0)
            {
                flag = 1;
                printf("commit hash: %s\n", tmp);
                fgets(temp, 100, file);
                if (*(temp + strlen(temp) - 1) == '\n')
                {
                    *(temp + strlen(temp) - 1) = '\0';
                }
                printf("%s\n", temp);
                while (fscanf(file, "%s", temp) != EOF)
                {
                    if (*(temp + strlen(temp) - 1) == '\n')
                    {
                        *(temp + strlen(temp) - 1) = '\0';
                    }
                    if (strcmp(temp, "time:") == 0)
                    {
                        break;
                    }
                }
                fgets(temp, 100, file);
                printf("time: %s", temp);
                fgets(temp, 100, file);
                printf("%s", temp);
                fgets(temp, 100, file);
                printf("%s", temp);
                fgets(temp, 100, file);
                printf("%s", temp);
                fgets(temp, 100, file);
                printf("%s\n", temp);
                printf(".......................................\n");
            }
        }
        else if (strcmp(mode, "-before") == 0)
        {
            if (status == 1 || status == 0)
            {
                flag = 1;
                printf("commit hash: %s\n", tmp);
                fgets(temp, 100, file);
                if (*(temp + strlen(temp) - 1) == '\n')
                {
                    *(temp + strlen(temp) - 1) = '\0';
                }
                printf("%s\n", temp);
                while (fscanf(file, "%s", temp) != EOF)
                {
                    if (*(temp + strlen(temp) - 1) == '\n')
                    {
                        *(temp + strlen(temp) - 1) = '\0';
                    }
                    if (strcmp(temp, "time:") == 0)
                    {
                        break;
                    }
                }
                fgets(temp, 100, file);
                printf("time: %s", temp);
                fgets(temp, 100, file);
                printf("%s", temp);
                fgets(temp, 100, file);
                printf("%s", temp);
                fgets(temp, 100, file);
                printf("%s", temp);
                fgets(temp, 100, file);
                printf("%s\n", temp);
                printf(".......................................\n");
            }
        }
        fclose(file);
    }
    if (flag == 0)
    {
        printf("there is no commits in this spare of time");
    }
}

void branch_maker(char *branch_name)
{
    char *address = (char *)malloc(200 * sizeof(char));
    address = where_before();
    strcat(address, "/.neogit/branches");
    chdir(address);
    DIR *dir = opendir(".");
    struct dirent *entry;
    int flag = 0;
    while ((entry = readdir(dir)) != NULL)
    {
        if (strcmp(entry->d_name, branch_name) == 0)
        {
            flag = 1;
        }
    }
    if (flag == 1)
        printf("this branch has already added");
    else
    {
        mkdir(branch_name, 0755);
    }
}

void branch_show()
{
    char *address = (char *)malloc(100 * sizeof(char));
    address = where_before();
    strcat(address, "/.neogit/branches");
    chdir(address);
    struct dirent *entry;
    DIR *dir = opendir(".");
    printf("branches list:\n");
    while ((entry = readdir(dir)) != NULL)
    {
        if (strcmp(entry->d_name, ".") != 0 && strcmp(entry->d_name, "..") != 0 && strcmp(entry->d_name, ".DS_Store") != 0)
            printf("%s\n", entry->d_name);
    }
}

void apply_checkout(char *number)
{

    char *address = (char *)malloc(200 * sizeof(char));
    address = where_before();
    strcat(address, "/.neogit/files");
    chdir(address);
    DIR *dir = opendir(".");
    struct dirent *entry;
    while ((entry = readdir(dir)) != NULL)
    {
        if (strcmp(entry->d_name, ".") != 0 && strcmp(entry->d_name, "..") != 0 && strcmp(entry->d_name, ".DS_Store") != 0)
        {
            char *a = open_directory(entry->d_name, number);
            if (a == NULL)
            {
                continue;
            }
            char *address2 = (char *)malloc(200 * sizeof(char));
            char cwd[1000];
            getcwd(cwd, sizeof(cwd));
            address2 = where_before();
            strcat(address2, "/.neogit/files/");
            strcat(address2, a);
            strcat(address2, "/");
            strcat(address2, number);
            chdir(cwd);
            for (int i = 0; i < strlen(a); i++)
            {
                if (*(a + i) == '-')
                {
                    *(a + i) = '/';
                }
            }
            char *destination = (char *)malloc(200 * sizeof(char));
            getcwd(cwd, sizeof(cwd));
            destination = where_before();
            chdir(cwd);
            strcat(destination, "/");
            strcat(destination, a);
            char *command = (char *)malloc(200 * sizeof(char));
            strcpy(command, "cp ");
            strcat(command, address2);
            strcat(command, " ");
            strcat(command, destination);
            system(command);
        }
    }
}

char *open_directory(char *name, char *number)
{
    DIR *dir = opendir(name);
    struct dirent *entry;
    while ((entry = readdir(dir)) != NULL)
    {
        if (strcmp(entry->d_name, number) == 0)
        {
            return name;
        }
    }
    return NULL;
}

void checkout_to_branch(char *branch_name)
{
    char *address2 = where_before();
    chdir(address2);
    chdir(".neogit");
    FILE *file = fopen("staging", "r");
    if (fscanf(file, "%s", address2) != EOF)
    {
        printf("your staging area is not empty yet");
        return;
    }
    char *address = (char *)malloc(200 * sizeof(char));
    address = where_before();
    strcat(address, "/.neogit/branches");
    chdir(address);
    DIR *dir = opendir(".");
    struct dirent *entry;
    int flag = 0;
    while ((entry = readdir(dir)) != NULL)
    {
        if (strcmp(branch_name, entry->d_name) == 0)
        {
            flag = 1;
        }
    }
    if (flag == 0)
    {
        printf("there is no branch with this name");
        return;
    }
    closedir(dir);
    strcat(address, "/");
    strcat(address, branch_name);
    chdir(address);
    dir = opendir(".");
    int files[50];
    for (int i = 0; i < 50; i++)
    {
        files[i] = 0;
    }
    int counter = 0;

    while ((entry = readdir(dir)) != NULL)
    {
        if (strcmp(entry->d_name, ".") != 0 && strcmp(entry->d_name, "..") != 0 && strcmp(entry->d_name, ".DS_Store") != 0)
        {
            sscanf(entry->d_name, "%d", &files[counter]);
            counter++;
        }
    }
    for (int i = 0; i < counter; i++)
    {
        for (int j = i; j < counter; j++)
        {
            if (files[i] > files[j])
            {
                int x = files[i];
                files[i] = files[j];
                files[j] = x;
            }
        }
    }
    for (int i = 0; i < counter; i++)
    {
        char temp3[5];
        sprintf(temp3, "%d", files[i]);
        apply_checkout(temp3);
    }
    // changing the config file data:
    // branch name:
    address = where_before();
    chdir(address);
    chdir(".neogit");
    FILE *file2 = fopen("config", "r");
    FILE *file3 = fopen("tmp", "w");
    for (int i = 0; i < 4; i++)
    {
        fgets(address, 200, file2);
        fprintf(file3, "%s", address);
    }
    fprintf(file3, "branch: %s", branch_name);
    remove("config");
    rename("tmp", "config");
    // current hash name:
    char *add = where_before();
    chdir(add);
    chdir(".neogit");
    chdir("branches");
    chdir(branch_name);
    DIR *dir4 = opendir(".");
    int max = 0;
    while ((entry = readdir(dir4)) != NULL)
    {
        if (strcmp(entry->d_name, ".") != 0 && strcmp(entry->d_name, "..") != 0 && strcmp(entry->d_name, ".DS_Store") != 0)
        {
            int numb = atoi(entry->d_name);
            if (max < numb)
            {
                max = numb;
            }
        }
    }
    char reee[21];
    sprintf(reee, "%d", max);
    int number = how_many_commits();
    // printf("number = %d\n", number);
    // printf("max = %d\n", max);

    // if (number == max)
    // {
    //     HEAD_status_to_zero();
    // }
    // else
    // {
    //     HEAD_status_to_one();
    // }
}

void checkout_to_hash(char *commit_hash)
{
    char *address = where_before();
    int commit_id = atoi(commit_hash);
    chdir(address);
    chdir(".neogit");
    chdir("commits");
    int counter = 0;
    DIR *dir = opendir(".");
    struct dirent *entry;
    while ((entry = readdir(dir)) != NULL)
    {
        if (strcmp(entry->d_name, ".") != 0 && strcmp(entry->d_name, "..") != 0 && strcmp(entry->d_name, ".DS_Store") != 0)
        {
            counter++;
        }
    }
    if (commit_id > counter)
    {
        printf("there is no commit with this hash");
        return;
    }
    for (int i = 1; i <= commit_id; i++)
    {
        char temp[5];
        sprintf(temp, "%d", i);
        apply_checkout(temp);
    }
    char *add = where_before();
    chdir(add);
    chdir(".neogit");
    FILE *file = fopen("config", "r");
    for (int i = 0; i < 2; i++)
    {
        fgets(add, 100, file);
    }
    fscanf(file, "%s", add);
    fscanf(file, "%s", add);
    if (strcmp(add, commit_hash) != 0)
    {
        FILE *file2 = fopen("HEAD_status", "w");
        fprintf(file2, "1");
        fclose(file2);
    }
    else
    {
        FILE *file2 = fopen("HEAD_status", "w");
        fprintf(file2, "0");
        fclose(file2);
    }
}

int how_many_commits()
{
    char *address = where_before();
    chdir(address);
    chdir(".neogit");
    chdir("commits");
    int counter = 0;
    DIR *dir = opendir(".");
    struct dirent *entry;
    while ((entry = readdir(dir)) != NULL)
    {
        if (strcmp(entry->d_name, ".") != 0 && strcmp(entry->d_name, "..") != 0 && strcmp(entry->d_name, ".DS_Store") != 0)
        {
            counter++;
        }
    }
    return counter;
}

void HEAD_status_to_zero()
{
    char *address = where_before();
    chdir(address);
    chdir(".neogit");
    FILE *file = fopen("HEAD_status", "w");
    fprintf(file, "0");
    fclose(file);
}

void HEAD_status_to_one()
{
    char *address = where_before();
    chdir(address);
    chdir(".neogit");
    FILE *file = fopen("HEAD_status", "w");
    fprintf(file, "1");
    fclose(file);
}

void branch_log(char *branch_name)
{
    char *address = where_before();
    chdir(address);
    chdir(".neogit/commits");
    DIR *dir = opendir(".");
    struct dirent *entry;
    int counter = 0;
    while ((entry = readdir(dir)) != NULL)
    {
        if (strcmp(entry->d_name, ".") != 0 && strcmp(entry->d_name, "..") != 0 && strcmp(entry->d_name, ".DS_Store") != 0)
        {
            counter++;
        }
    }
    for (int i = counter; i > 0; i--)
    {
        char temp7[10];
        sprintf(temp7, "%d", i);
        FILE *file = fopen(temp7, "r");
        char *temp = (char *)malloc(100 * sizeof(char));
        while (fscanf(file, "%s", temp) != EOF)
        {
            if (strcmp(temp, "branch:") == 0)
            {
                break;
            }
        }
        fscanf(file, "%s", temp);
        if (strcmp(temp, branch_name) == 0)
        {
            fseek(file, 0, SEEK_SET);
            printf("commit hash: %s\n", temp7);
            fgets(temp, 100, file);
            printf("%s", temp);
            while (fscanf(file, "%s", temp) != EOF)
            {
                if (strcmp(temp, "time:") == 0)
                {
                    break;
                }
            }
            fgets(temp, 100, file);
            printf("time: %s", temp);
            fgets(temp, 100, file);
            printf("%s", temp);
            fgets(temp, 100, file);
            printf("%s", temp);
            fgets(temp, 100, file);
            printf("%s", temp);
            fgets(temp, 100, file);
            printf("%s\n", temp);
            printf(".......................................\n");
        }
    }
}

char *back_slash_n_remover(char *string)
{
    for (int i = 0; i < strlen(string); i++)
    {
        if (*(string + i) == '\n')
        {
            *(string + i) = '\0';
        }
    }
    return string;
}

void tree()
{
    char *address = where_before();
    chdir(address);
    chdir(".neogit");
    chdir("branches");
    DIR *dir = opendir(".");
    struct dirent *entry;
    int counter = 0;
    char *other_branch_name = (char *)malloc(100 * sizeof(char));
    while ((entry = readdir(dir)) != NULL)
    {
        if (strcmp(entry->d_name, ".") != 0 && strcmp(entry->d_name, "..") != 0 && strcmp(entry->d_name, ".DS_Store") != 0)
        {
            counter++;
            if (strcmp(entry->d_name, "master") != 0)
            {
                strcpy(other_branch_name, entry->d_name);
            }
        }
    }
    if (counter > 2)
    {
        printf("you have more than 2 branches");
        return;
    }
    else if (counter == 1)
    {
        char *address2 = where_before();
        chdir(address2);
        chdir(".neogit/commits");
        int counter2 = 0;
        dir = opendir(".");
        while ((entry = readdir(dir)) != NULL)
        {
            if (strcmp(entry->d_name, ".") != 0 && strcmp(entry->d_name, "..") != 0 && strcmp(entry->d_name, ".DS_Store") != 0)
            {
                counter2++;
            }
        }
        for (int i = 1; i < counter2; i++)
        {
            printf("%d\n|\n", i);
        }
        printf("%d\n", counter2);
        return;
    }
    else if (counter == 2)
    {
        char *address2 = where_before();
        chdir(address2);
        chdir(".neogit/branches");
        int *master_branch = (int *)malloc(100 * sizeof(int));
        int *second_branch = (int *)malloc(100 * sizeof(int));
        // finding names of txt files
        chdir("master");
        dir = opendir(".");
        int counter1 = 0;
        while ((entry = readdir(dir)) != NULL)
        {
            if (strcmp(entry->d_name, ".") != 0 && strcmp(entry->d_name, "..") != 0 && strcmp(entry->d_name, ".DS_Store") != 0)
            {
                int number = atoi(entry->d_name);
                *(master_branch + counter1) = number;
                counter1++;
            }
        }
        int counter2 = 0;
        chdir("..");
        chdir(other_branch_name);
        dir = opendir(".");
        while ((entry = readdir(dir)) != NULL)
        {
            if (strcmp(entry->d_name, ".") != 0 && strcmp(entry->d_name, "..") != 0 && strcmp(entry->d_name, ".DS_Store") != 0)
            {
                int number = atoi(entry->d_name);
                *(second_branch + counter2) = number;
                counter2++;
            }
        }
        for (int i = 0; i < counter1; i++)
        {
            for (int j = i; j < counter1; j++)
            {
                if (*(master_branch + i) > *(master_branch + j))
                {
                    int temp4 = *(master_branch + i);
                    *(master_branch + i) = *(master_branch + j);
                    *(master_branch + j) = temp4;
                }
            }
        }
        for (int i = 0; i < counter2; i++)
        {
            for (int j = i; j < counter2; j++)
            {
                if (*(second_branch + i) > *(second_branch + j))
                {
                    int temp4 = *(second_branch + i);
                    *(second_branch + i) = *(second_branch + j);
                    *(second_branch + j) = temp4;
                }
            }
        }
        // finish
        // remember: counter1 ---> master_branch / counter2 ---> second_branch
        // real tree:
        int alaki = *(second_branch + 0);
        for (int i = 1; i < alaki - 1; i++)
        {
            printf("%d\n|\n", i);
        }
        printf("%d\n|", alaki - 1);
        printf(" \\\n");
        int whereInFirst = alaki - 1;
        int whereInSecond = 0;

        while (1)
        {
            if (whereInFirst < counter1)
                printf("%d ", *(master_branch + whereInFirst));
            if (whereInSecond < counter2)
                printf("%d ", *(second_branch + whereInSecond));
            printf("\n");
            if (whereInFirst + 1 < counter1)
                printf("| ");
            if (whereInSecond + 1 < counter2)
                printf("| ");
            printf("\n");
            whereInFirst++;
            whereInSecond++;
            if (whereInFirst >= counter1 && whereInSecond >= counter2)
                break;
        }
    }
}

void search_log(char *word, int number)
{
    char *address = where_before();
    chdir(address);
    chdir(".neogit");
    chdir("commits");
    char temp[7];
    sprintf(temp, "%d", number);
    FILE *file = fopen(temp, "r");
    char *data = (char *)malloc(100 * sizeof(char));
    fscanf(file, "%s", data);
    fgets(data, 1000, file);
    if (*(data + strlen(data) - 1) == '\n')
    {
        *(data + strlen(data) - 1) = '\0';
    }
    char *check = (char *)malloc(100 * sizeof(char));
    check = strtok(data, " ");
    if (strcmp(check, word) == 0)
    {
        fseek(file, 0, SEEK_SET);
        printf("commit hash: %d\n", number);
        fgets(check, 100, file);
        printf("%s", check);
        while (fscanf(file, "%s", check) != EOF)
        {
            if (strcmp(check, "time:") == 0)
            {
                break;
            }
        }
        fgets(check, 100, file);
        printf("time: %s", check);
        fgets(check, 100, file);
        printf("%s", check);
        fgets(check, 100, file);
        printf("%s", check);
        fgets(check, 100, file);
        printf("%s", check);
        fgets(check, 100, file);
        printf("%s\n", check);
        printf(".......................................\n");
    }
    while ((check = strtok(NULL, " ")) != NULL)
    {
        if (strcmp(check, word) == 0)
        {
            fseek(file, 0, SEEK_SET);
            printf("commit hash: %d\n", number);
            fgets(check, 100, file);
            printf("%s", check);
            while (fscanf(file, "%s", check) != EOF)
            {
                if (strcmp(check, "time:") == 0)
                {
                    break;
                }
            }
            fgets(check, 100, file);
            printf("time: %s", check);
            fgets(check, 100, file);
            printf("%s", check);
            fgets(check, 100, file);
            printf("%s", check);
            fgets(check, 100, file);
            printf("%s", check);
            fgets(check, 100, file);
            printf("%s\n", check);
            printf(".......................................\n");
        }
    }
}

void tag_handler(char *const argv[], int argc)
{
    if (strcmp(argv[2], "-a") != 0)
    {
        printf("enter a valid command");
    }
    else
    {
        int last_commit = how_many_commits();
        char *address = where_before();
        chdir(address);
        chdir(".neogit");
        FILE *myfile = fopen("config", "r");
        char *name = (char *)malloc(30 * sizeof(char));
        fscanf(myfile, "%s", name);
        fgets(name, 100, myfile);
        if (*(name + strlen(name) - 1) == '\n')
        {
            *(name + strlen(name) - 1) = '\0';
        }
        // name is handled
        //
        chdir(".neogit/tags");
        char temp[7];
        sprintf(temp, "%d", last_commit);
        // temp = commit_id
        char *time = time_finder();
        // time is handled
        struct dirent *entry;
        int flag = 0;
        char *address2 = where_before();
        strcat(address2, "/.neogit/tags");
        chdir(address2);
        DIR *dir = opendir(".");
        while ((entry = readdir(dir)) != NULL)
        {
            if (strcmp(argv[3], entry->d_name) == 0)
            {
                flag = 1;
            }
        }
        closedir(dir);
        if (argc == 4)
        {
            if (flag == 1)
            {
                printf("this tag name is already in use");
                return;
            }
            FILE *file = fopen(argv[3], "w");
            fprintf(file, "tag_name: %s\n", argv[3]);
            fprintf(file, "commit_id: %s\n", temp);
            fprintf(file, "author: %s\n", name);
            fprintf(file, "time: %s\n", time);
            fclose(file);
            return;
        }
        else if (argc == 5 && strcmp(argv[argc - 1], "-f") == 0)
        {
            FILE *file = fopen(argv[3], "w");
            fprintf(file, "tag_name: %s\n", argv[3]);
            fprintf(file, "commit_id: %s\n", temp);
            fprintf(file, "author: %s\n", name);
            fprintf(file, "time: %s\n", time);
            fclose(file);
            return;
        }
        else if (argc == 6 && strcmp(argv[4], "-m") == 0)
        {
            if (flag == 1)
            {
                printf("this tag name is already in use");
                return;
            }
            FILE *file = fopen(argv[3], "w");
            fprintf(file, "tag_name: %s\n", argv[3]);
            fprintf(file, "commit_id: %s\n", temp);
            fprintf(file, "author: %s\n", name);
            fprintf(file, "time: %s\n", time);
            fprintf(file, "message: %s\n", argv[5]);
            fclose(file);
            return;
        }
        else if (argc == 7 && strcmp(argv[argc - 1], "-f") == 0)
        {
            FILE *file = fopen(argv[3], "w");
            fprintf(file, "tag_name: %s\n", argv[3]);
            fprintf(file, "commit_id: %s\n", temp);
            fprintf(file, "author: %s\n", name);
            fprintf(file, "time: %s\n", time);
            fprintf(file, "message: %s\n", argv[5]);
            fclose(file);
            return;
        }
    }
}

void tag_shower()
{
    char *address = where_before();
    strcat(address, "/.neogit/tags");
    chdir(address);
    DIR *dir = opendir(".");
    struct dirent *entry;
    char names[30][100];
    int counter = 0;
    while ((entry = readdir(dir)) != NULL)
    {
        if (strcmp(entry->d_name, ".") != 0 && strcmp(entry->d_name, "..") != 0 && strcmp(entry->d_name, ".DS_Store") != 0)
        {
            strcpy(names[counter], entry->d_name);
            counter++;
        }
    }
    for (int i = 0; i < counter; i++)
    {
        for (int j = i; j < counter; j++)
        {
            if (strcmp(names[i], names[j]) > 0)
            {
                char temp[100];
                strcpy(temp, names[i]);
                strcpy(names[i], names[j]);
                strcpy(names[j], temp);
            }
        }
    }
    for (int i = 0; i < counter; i++)
    {
        printf("%s\n", names[i]);
    }
}

void tag_log(char *tag_name)
{
    char *address = where_before();
    strcat(address, "/.neogit/tags");
    chdir(address);
    DIR *dir = opendir(".");
    struct dirent *entry;
    int flag = 0;
    while ((entry = readdir(dir)) != NULL)
    {
        if (strcmp(entry->d_name, ".") != 0 && strcmp(entry->d_name, "..") != 0 && strcmp(entry->d_name, ".DS_Store") != 0)
        {
            if (strcmp(entry->d_name, tag_name) == 0)
            {
                flag = 1;
            }
        }
    }
    if (flag == 0)
    {
        printf("this flag does not exist");
        return;
    }
    else
    {
        FILE *file = fopen(tag_name, "r");
        char *temp = (char *)malloc(100 * sizeof(char));
        while (fgets(temp, 100, file) != NULL)
        {
            printf("%s", temp);
        }
    }
}

int isItonlyWhiteSpace(char *string)
{
    int flag = 0;
    for (int i = 0; i < strlen(string); i++)
    {
        if (*(string + i) != ' ' && *(string + i) != '\t' && *(string + i) != '\0' && *(string + i) != '\n')
        {
            flag = 1;
        }
    }
    return flag;
}

char *sentence_corrector(char *temp)
{
    char *result = (char *)malloc(100 * sizeof(char));
    int counter = 0;
    if (temp == NULL)
    {
        printf("1");
        return NULL;
    }

    for (int i = 0; i < strlen(temp); i++)
    {
        if (*(temp + i) == '\t')
        {
            *(temp + i) = ' ';
        }
    }

    if (temp == NULL)
    {
        printf("2");
        return NULL;
    }
    for (int i = 0; i < strlen(temp) - 1; i++)
    {
        if (*(temp + i) == ' ' && *(temp + i + 1) == ' ')
        {
            continue;
        }
        *(result + counter) = *(temp + i);
        counter++;
    }
    if (result == NULL)
    {
        printf("4");
        return NULL;
    }
    *(result + counter) = *(temp + strlen(temp) - 1);
    if (result == NULL)
    {
        printf("5");
        return NULL;
    }

    if (*(result + 0) == ' ')
    {
        for (int i = 0; i < strlen(temp); i++)
        {
            *(result + i) = *(result + i + 1);
        }
    }
    if (result == NULL)
    {
        printf("6");
        return NULL;
    }
    return result;
}

void file_changer(char *file_name, char *output_name)
{
    FILE *file = fopen(file_name, "r");
    FILE *file1 = fopen(output_name, "w");
    char *temp = (char *)malloc(200 * sizeof(char));
    char words[100][100];
    int n = 0;
    while (fgets(temp, 200, file) != NULL)
    {
        if (*(temp + strlen(temp) - 1) == '\n')
            *(temp + strlen(temp) - 1) = '\0';
        if (!isItonlyWhiteSpace(temp))
        {
            continue;
        }

        strcpy(words[n], temp);
        n++;
    }
    char *tempp = (char *)malloc(100 * sizeof(char));
    for (int i = 0; i < n; i++)
    {
        tempp = sentence_corrector(words[i]);
        fprintf(file1, "%s\n", tempp);
    }
    fclose(file);
    fclose(file1);
}

void diff(char *first_file, char *second_file)
{
    file_changer(first_file, "1");
    file_changer(second_file, "2");
    FILE *file1 = fopen("1", "r");
    FILE *file2 = fopen("2", "r");
    char *temp1 = (char *)malloc(100 * sizeof(char));
    char *temp2 = (char *)malloc(100 * sizeof(char));
    int counter = 0;
    while (fgets(temp1, 1000, file1) != NULL && fgets(temp2, 1000, file2) != NULL)
    {
        counter++;
        if (strcmp(temp1, temp2) != 0)
        {
            printf("<<<<<\n");
            printf("%s-%d\n", first_file, counter);
            print_color(31, temp1);
            printf("%s-%d\n", second_file, counter);
            print_color(32, temp2);
            printf(">>>>>\n");
        }
    }
    remove("1");
    remove("2");
    fclose(file1);
    fclose(file2);
}

void print_color(int color_code, const char *text)
{
    printf("\033[1;%dm%s\033[0m", color_code, text);
}

void diff_line(char *first_file, char *second_file, int first_line, int second_line)
{
    file_changer(first_file, "1");
    file_changer(second_file, "2");
    FILE *file1 = fopen("1", "r");
    FILE *file2 = fopen("2", "r");
    char *temp1 = (char *)malloc(100 * sizeof(char));
    char *temp2 = (char *)malloc(100 * sizeof(char));
    int counter = 0;
    while (fgets(temp1, 1000, file1) != NULL && fgets(temp2, 1000, file2) != NULL)
    {

        counter++;
        if (counter >= first_line && counter <= second_line)
        {
            if (strcmp(temp1, temp2) != 0)
            {
                printf("<<<<<\n");
                printf("%s-%d\n", first_file, counter);
                print_color(31, temp1);
                printf("%s-%d\n", second_file, counter);
                print_color(32, temp2);
                printf(">>>>>\n");
            }
        }
    }
    remove("1");
    remove("2");
    fclose(file1);
    fclose(file2);
}

void grep(char *file_name, char *word)
{
    file_changer(file_name, "-");
    FILE *file = fopen("-", "r");
    char *temp = (char *)malloc(100 * sizeof(char));
    char *word_iterator = (char *)malloc(100 * sizeof(char));
    char *full_sentence = (char *)malloc(100 * sizeof(char));
    fseek(file, 0, SEEK_SET);
    while (fgets(temp, 200, file) != NULL)
    {
        if (*(temp + strlen(temp) - 1) == '\n')
        {
            *(temp + strlen(temp) - 1) = '\0';
        }
        strcpy(full_sentence, temp);
        word_iterator = strtok(temp, " ");
        if (word_iterator == NULL)
        {
            continue;
        }
        if (strcmp(word_iterator, word) == 0)
        {
            printf("%s\n", full_sentence);
        }
        while (1)
        {
            word_iterator = strtok(NULL, " ");
            if (word_iterator == NULL)
            {
                break;
            }
            if (strcmp(word_iterator, word) == 0)
            {
                printf("%s\n", full_sentence);
                break;
            }
        }
    }
    fclose(file);
    remove("-");
}

void grep_n(char *file_name, char *word)
{
    int counter = 0;
    file_changer(file_name, "1");
    FILE *file = fopen("1", "r");
    char *temp = (char *)malloc(100 * sizeof(char));
    char *word_iterator = (char *)malloc(100 * sizeof(char));
    char *full_sentence = (char *)malloc(100 * sizeof(char));
    fseek(file, 0, SEEK_SET);
    while (fgets(temp, 200, file) != NULL)
    {
        counter++;
        if (*(temp + strlen(temp) - 1) == '\n')
        {
            *(temp + strlen(temp) - 1) = '\0';
        }
        strcpy(full_sentence, temp);
        word_iterator = strtok(temp, " ");
        if (word_iterator == NULL)
        {
            continue;
        }
        if (strcmp(word_iterator, word) == 0)
        {
            printf("sentence number: %d\n%s\n", counter, full_sentence);
            printf("....................\n");
        }
        while (1)
        {
            word_iterator = strtok(NULL, " ");
            if (word_iterator == NULL)
            {
                break;
            }
            if (strcmp(word_iterator, word) == 0)
            {
                printf("sentence number: %d\n%s\n", counter, full_sentence);
                printf("....................\n");
                break;
            }
        }
    }
    fclose(file);
    remove("1");
}

void grep_commit(int commit_id, char *word)
{
    char commit[10];
    sprintf(commit, "%d", commit_id);
    char *address = where_before();
    chdir(address);
    chdir(".neogit/files");
    DIR *dir = opendir(".");
    struct dirent *entry;

    while ((entry = readdir(dir)) != NULL)
    {
        if (strcmp(entry->d_name, ".") != 0 && strcmp(entry->d_name, "..") != 0 && strcmp(entry->d_name, ".DS_Store") != 0)
        {

            chdir(entry->d_name);
            if (access(commit, F_OK) == 0)
            {
                grep(commit, word);
            }
            chdir("..");
        }
    }
}

void grep_commit_n(int commit_id, char *word)
{
    char commit[10];
    sprintf(commit, "%d", commit_id);
    char *address = where_before();
    chdir(address);
    chdir(".neogit/files");
    DIR *dir = opendir(".");
    struct dirent *entry;

    while ((entry = readdir(dir)) != NULL)
    {
        if (strcmp(entry->d_name, ".") != 0 && strcmp(entry->d_name, "..") != 0 && strcmp(entry->d_name, ".DS_Store") != 0)
        {

            chdir(entry->d_name);
            if (access(commit, F_OK) == 0)
            {
                grep_n(commit, word);
            }
            chdir("..");
        }
    }
}

char *getFileExtension(const char *filename)
{
    char *extension = strrchr(filename, '.');

    if (extension == NULL)
    {
        return NULL;
    }
    else
    {
        return extension + 1;
    }
}

char *slash_remover(char *string)
{
    char *temp = (char *)malloc(100 * sizeof(char));
    string = strtok(string, "/");
    strcpy(temp, string);
    while (1)
    {
        string = strtok(NULL, "/");
        if (string == NULL)
        {
            break;
        }
        strcpy(temp, string);
    }
    return temp;
}

int todo_check(char *filename)
{
    FILE *file = fopen(filename, "r");
    filename = slash_remover(filename);

    char *extention = getFileExtension(filename);
    if (strcmp(extention, "c") == 0 || strcmp(extention, "cpp") == 0 || strcmp(extention, "txt") == 0)
    {
        char *todo = (char *)malloc(10 * sizeof(char));
        strcpy(todo, "TODO");
        while (fscanf(file, "%s", extention) != EOF)
        {
            if (strcmp(extention, todo) == 0)
            {
                return -1;
            }
        }
        return 1;
    }
    else
    {
        fclose(file);
        return 0;
    }
    fclose(file);
}

void todo_full_check()
{
    chdir("staging_area");
    DIR *dir = opendir(".");
    struct dirent *entry;
    char cwd[1000];
    getcwd(cwd, sizeof(cwd));
    char *address = where_before();
    char *name = (char *)malloc(100 * sizeof(char));
    printf("todo-check hook:\n");
    char *ttt = where_before();
    chdir(ttt);
    chdir(".neogit");
    FILE *fil1 = fopen("precommit_result", "a");
    chdir(cwd);
    while ((entry = readdir(dir)) != NULL)
    {
        if (strcmp(entry->d_name, ".") != 0 && strcmp(entry->d_name, "..") != 0 && strcmp(entry->d_name, ".DS_Store") != 0)
        {
            strcpy(name, entry->d_name);
            for (int i = 0; i < strlen(name); i++)
            {
                if (*(name + i) == '-')
                {
                    *(name + i) = '/';
                }
            }

            address = where_before();
            strcat(address, "/");
            strcat(address, name);
            int result = todo_check(address);
            chdir(cwd);
            if (result == 0)
            {
                char *u = (char *)malloc(100 * sizeof(char));
                strcpy(u, entry->d_name);
                for (int i = 0; i < strlen(u); i++)
                {
                    if (*(u + strlen(u) - 1) == '-')
                    {
                        *(u + strlen(u) - 1) = '/';
                    }
                }
                printf("%s....................", u);
                print_color(33, "skipped");
                printf("\n");
            }
            else if (result == 1)
            {
                char *u = (char *)malloc(100 * sizeof(char));
                strcpy(u, entry->d_name);
                for (int i = 0; i < strlen(u); i++)
                {
                    if (*(u + strlen(u) - 1) == '-')
                    {
                        *(u + strlen(u) - 1) = '/';
                    }
                }
                printf("%s....................", u);
                print_color(32, "passed");
                printf("\n");
            }
            else if (result == -1)
            {
                char *u = (char *)malloc(100 * sizeof(char));
                strcpy(u, entry->d_name);
                for (int i = 0; i < strlen(u); i++)
                {
                    if (*(u + strlen(u) - 1) == '-')
                    {
                        *(u + strlen(u) - 1) = '/';
                    }
                }
                printf("%s....................", u);
                print_color(31, "failed");
                printf("\n");
                fprintf(fil1, "%s\n", u);
            }
        }
    }
    chdir("..");
}

int balance_braces(char *filename)
{
    FILE *file = fopen(filename, "r");
    filename = slash_remover(filename);
    char *extention = getFileExtension(filename);
    if (strcmp(extention, "c") == 0 || strcmp(extention, "cpp") == 0 || strcmp(extention, "txt") == 0)
    {
        char temp;
        int parantez_right = 0;
        int croshe_right = 0;
        int bracket_rigth = 0;
        int parantez_left = 0;
        int croshe_left = 0;
        int bracket_left = 0;
        while ((temp = fgetc(file)) != EOF)
        {
            if (temp == '(')
            {
                parantez_left++;
            }
            if (temp == ')')
            {
                parantez_right++;
            }
            if (temp == '[')
            {
                bracket_left++;
            }
            if (temp == ']')
            {
                bracket_rigth++;
            }
            if (temp == '{')
            {
                croshe_left++;
            }
            if (temp == '}')
            {
                croshe_right++;
            }
        }
        if (croshe_left == croshe_right && bracket_left == bracket_rigth && parantez_left == parantez_right)
        {
            return 1;
        }
        else
        {
            return -1;
        }
    }
    else
    {
        return 0;
    }
}

void balance_braces_full()
{
    chdir("staging_area");
    DIR *dir = opendir(".");
    struct dirent *entry;
    char cwd[1000];
    getcwd(cwd, sizeof(cwd));
    char *address = where_before();
    char *name = (char *)malloc(100 * sizeof(char));
    printf("balance-braces hook:\n");
    char *ttt = where_before();
    chdir(ttt);
    chdir(".neogit");
    FILE *fil1 = fopen("precommit_result", "a");
    chdir(cwd);
    while ((entry = readdir(dir)) != NULL)
    {
        if (strcmp(entry->d_name, ".") != 0 && strcmp(entry->d_name, "..") != 0 && strcmp(entry->d_name, ".DS_Store") != 0)
        {
            strcpy(name, entry->d_name);
            for (int i = 0; i < strlen(name); i++)
            {
                if (*(name + i) == '-')
                {
                    *(name + i) = '/';
                }
            }
            address = where_before();
            strcat(address, "/");
            strcat(address, name);
            int result = balance_braces(address);
            chdir(cwd);
            if (result == 0)
            {
                char *u = (char *)malloc(100 * sizeof(char));
                strcpy(u, entry->d_name);
                for (int i = 0; i < strlen(u); i++)
                {
                    if (*(u + strlen(u) - 1) == '-')
                    {
                        *(u + strlen(u) - 1) = '/';
                    }
                }
                printf("%s....................", u);
                print_color(33, "skipped");
                printf("\n");
            }
            else if (result == 1)
            {
                char *u = (char *)malloc(100 * sizeof(char));
                strcpy(u, entry->d_name);
                for (int i = 0; i < strlen(u); i++)
                {
                    if (*(u + strlen(u) - 1) == '-')
                    {
                        *(u + strlen(u) - 1) = '/';
                    }
                }
                printf("%s....................", u);
                print_color(32, "passed");
                printf("\n");
            }
            else if (result == -1)
            {
                char *u = (char *)malloc(100 * sizeof(char));
                strcpy(u, entry->d_name);
                for (int i = 0; i < strlen(u); i++)
                {
                    if (*(u + strlen(u) - 1) == '-')
                    {
                        *(u + strlen(u) - 1) = '/';
                    }
                }
                printf("%s....................", u);
                print_color(31, "failed");
                printf("\n");
                fprintf(fil1, "%s\n", u);
            }
        }
    }
    chdir("..");
}

int character_limit(char *filename)
{
    FILE *file = fopen(filename, "r");
    filename = slash_remover(filename);
    char *extention = getFileExtension(filename);
    int counter = 0;
    char temp;
    if (strcmp(extention, "c") == 0 || strcmp(extention, "cpp") == 0 || strcmp(extention, "txt") == 0)
    {
        while ((temp = fgetc(file)) != EOF)
        {
            counter++;
        }
        if (counter > 20000)
        {
            return -1;
        }
        return 1;
    }
    else
    {
        return 0;
    }
}

void character_limit_full()
{
    chdir("staging_area");
    DIR *dir = opendir(".");
    struct dirent *entry;
    char cwd[1000];
    getcwd(cwd, sizeof(cwd));
    char *address = where_before();
    char *name = (char *)malloc(100 * sizeof(char));
    printf("charecters limit hook:\n");
    char *ttt = where_before();
    chdir(ttt);
    chdir(".neogit");
    FILE *fil1 = fopen("precommit_result", "a");
    chdir(cwd);
    while ((entry = readdir(dir)) != NULL)
    {
        if (strcmp(entry->d_name, ".") != 0 && strcmp(entry->d_name, "..") != 0 && strcmp(entry->d_name, ".DS_Store") != 0)
        {
            strcpy(name, entry->d_name);
            for (int i = 0; i < strlen(name); i++)
            {
                if (*(name + i) == '-')
                {
                    *(name + i) = '/';
                }
            }
            address = where_before();
            strcat(address, "/");
            strcat(address, name);
            int result = character_limit(address);
            chdir(cwd);
            if (result == 0)
            {
                char *u = (char *)malloc(100 * sizeof(char));
                strcpy(u, entry->d_name);
                for (int i = 0; i < strlen(u); i++)
                {
                    if (*(u + strlen(u) - 1) == '-')
                    {
                        *(u + strlen(u) - 1) = '/';
                    }
                }
                printf("%s....................", u);
                print_color(33, "skipped");
                printf("\n");
            }
            else if (result == 1)
            {
                char *u = (char *)malloc(100 * sizeof(char));
                strcpy(u, entry->d_name);
                for (int i = 0; i < strlen(u); i++)
                {
                    if (*(u + strlen(u) - 1) == '-')
                    {
                        *(u + strlen(u) - 1) = '/';
                    }
                }
                printf("%s....................", u);
                print_color(32, "passed");
                printf("\n");
            }
            else if (result == -1)
            {
                char *u = (char *)malloc(100 * sizeof(char));
                strcpy(u, entry->d_name);
                for (int i = 0; i < strlen(u); i++)
                {
                    if (*(u + strlen(u) - 1) == '-')
                    {
                        *(u + strlen(u) - 1) = '/';
                    }
                }
                printf("%s....................", u);
                print_color(31, "failed");
                printf("\n");
                fprintf(fil1, "%s\n", u);
            }
        }
    }
    chdir("..");
}

long getFileSize(const char *filename)
{
    struct stat file_stat;

    if (stat(filename, &file_stat) == -1)
    {
        perror("Error getting file information");
        return -1;
    }

    return (long)file_stat.st_size;
}

void file_size_check_full()
{
    chdir("staging_area");
    DIR *dir = opendir(".");
    struct dirent *entry;
    char cwd[1000];
    getcwd(cwd, sizeof(cwd));
    char *address = where_before();
    char *name = (char *)malloc(100 * sizeof(char));
    printf("file size check hook:\n");
    char *ttt = where_before();
    chdir(ttt);
    chdir(".neogit");
    FILE *fil1 = fopen("precommit_result", "a");
    chdir(cwd);
    while ((entry = readdir(dir)) != NULL)
    {
        if (strcmp(entry->d_name, ".") != 0 && strcmp(entry->d_name, "..") != 0 && strcmp(entry->d_name, ".DS_Store") != 0)
        {
            strcpy(name, entry->d_name);
            for (int i = 0; i < strlen(name); i++)
            {
                if (*(name + i) == '-')
                {
                    *(name + i) = '/';
                }
            }
            address = where_before();
            strcat(address, "/");
            strcat(address, name);
            long result2 = getFileSize(address);
            int result;
            if (result2 > 5000000)
            {
                result = -1;
            }
            else if (result2 <= 5000000)
            {
                result = 1;
            }
            chdir(cwd);
            if (result == 0)
            {
                char *u = (char *)malloc(100 * sizeof(char));
                strcpy(u, entry->d_name);
                for (int i = 0; i < strlen(u); i++)
                {
                    if (*(u + strlen(u) - 1) == '-')
                    {
                        *(u + strlen(u) - 1) = '/';
                    }
                }
                printf("%s....................", u);
                print_color(33, "skipped");
                printf("\n");
            }
            else if (result == 1)
            {
                char *u = (char *)malloc(100 * sizeof(char));
                strcpy(u, entry->d_name);
                for (int i = 0; i < strlen(u); i++)
                {
                    if (*(u + strlen(u) - 1) == '-')
                    {
                        *(u + strlen(u) - 1) = '/';
                    }
                }
                printf("%s....................", u);
                print_color(32, "passed");
                printf("\n");
            }
            else if (result == -1)
            {
                char *u = (char *)malloc(100 * sizeof(char));
                strcpy(u, entry->d_name);
                for (int i = 0; i < strlen(u); i++)
                {
                    if (*(u + strlen(u) - 1) == '-')
                    {
                        *(u + strlen(u) - 1) = '/';
                    }
                }
                printf("%s....................", u);
                print_color(31, "failed");
                printf("\n");
                fprintf(fil1, "%s\n", u);
            }
        }
    }
    chdir("..");
}

void pre_commit()
{
    char *address = where_before();
    chdir(address);
    chdir(".neogit");
    FILE *file = fopen("precommit", "r");
    while (fgets(address, 100, file) != NULL)
    {
        if (*(address + strlen(address) - 1) == '\n')
        {
            *(address + strlen(address) - 1) = '\0';
        }
        if (strcmp(address, "todo-check") == 0)
        {
            todo_full_check();
        }
        if (strcmp(address, "balance-braces") == 0)
        {
            balance_braces_full();
        }
        if (strcmp(address, "charecter-limit") == 0)
        {
            character_limit_full();
        }
        if (strcmp(address, "file-size-check") == 0)
        {
            file_size_check_full();
        }
    }
    fclose(file);
}

int main(int argc, char *argv[])
{
    if (argc < 2)
    {
        fprintf(stdout, "please enter a valid command");
        return 1;
    }
    print_command(argc, argv);
    if (strcmp(argv[1], "config") == 0)
    {
        if (strcmp(argv[2], "--global") == 0)
        {
            if (strcmp(argv[3], "username") == 0 || strcmp(argv[3], "useremail") == 0)
            {
                chdir("/Users/arashghavami");
                DIR *dir = opendir(".");
                char *name = (char *)malloc(100 * sizeof(char));
                strcpy(name, argv[3]);
                strcat(name, ".txt");
                struct dirent *entry;
                while ((entry = readdir(dir)) != NULL)
                {
                    if (strcmp(entry->d_name, name) == 0)
                    {
                        remove(name);
                        break;
                    }
                }
                closedir(dir);
                for (int i = 4; i < argc; i++)
                {
                    int k = config_global(argv[3], argv[i]);
                }
            }
            else
            {
                printf("Enter a correct command (useremail/username)");
            }
        }
        else
        {
            char *full_data = (char *)malloc(100 * sizeof(char));
            strcpy(full_data, argv[3]);
            strcat(full_data, " ");
            for (int i = 4; i < argc; i++)
            {
                strcat(full_data, argv[i]);
                strcat(full_data, " ");
            }
            config_normal(argv[2], full_data);
        }
    }
    else if (strcmp(argv[1], "init") == 0)
    {
        char cwd[1024];
        if (getcwd(cwd, sizeof(cwd)) == NULL)
        {
            return 1;
        }
        chdir("/Users/arashghavami");
        DIR *dir = opendir(".");
        struct dirent *entry;
        int flag1 = 0;
        int flag2 = 0;
        while ((entry = readdir(dir)) != NULL)
        {
            if (strcmp("username.txt", entry->d_name) == 0)
            {
                flag1 = 1;
            }
            if (strcmp("useremail.txt", entry->d_name) == 0)
            {
                flag2 = 1;
            }
        }
        chdir(cwd);
        if (flag1 == 1 && flag2 == 1)
        {
            return run_init(argc, argv);
        }
        else
        {
            printf("you should config your username and useremail first");
            return 1;
        }
    }
    else if (strcmp(argv[1], "add") == 0)
    {
        char cwd[1000];
        getcwd(cwd, sizeof(cwd));
        char *address = (char *)malloc(1000 * sizeof(char));
        for (int i = 0; i < strlen(cwd); i++)
        {
            *(address + i) = cwd[i];
        }
        int x = strlen(address);
        if (strcmp(argv[2], "-n") != 0 && strcmp(argv[2], "-f") != 0)
        {
            for (int i = 2; i < argc; i++)
            {
                for (int i = x; i < strlen(address); i++)
                {
                    *(address + i) = '\0';
                }
                run_add(argv[i], address);
            }
        }
        if (strcmp(argv[2], "-f") == 0)
        {
            for (int i = 3; i < argc; i++)
            {
                for (int i = x; i < strlen(address); i++)
                {
                    *(address + i) = '\0';
                }
                run_add(argv[i], address);
            }
        }

        if (strcmp(argv[2], "-redo") == 0)
        {
            // redo
        }

        if (strcmp(argv[2], "-n") == 0)
        {
            int depth = (int)(argv[3][0] - '0');
            FILE *file = fopen(".add-n", "w");
            char *address = where_before();
            iterate_and_write_to_file(address, depth, file);
            fclose(file);
        }
    }
    else if (strcmp(argv[1], "reset") == 0)
    {
        if (strcmp(argv[2], "-f") != 0 && strcmp(argv[2], "-undo") != 0)
        {
            for (int i = 2; i < argc; i++)
            {
                run_reset(argc, argv[i]);
            }
        }
        if (strcmp(argv[2], "-f") == 0 && strcmp(argv[2], "-undo") != 0)
        {
            for (int i = 3; i < argc; i++)
            {
                run_reset(argc, argv[i]);
            }
        }
        if (strcmp(argv[2], "-undo") == 0)
        {
            reset_undo();
        }
    }
    else if (strcmp(argv[1], "commit") == 0)
    {
        if (strcmp(argv[2], "-m") == 0)
        {
            return run_commit(argc, argv);
        }
        else if (strcmp(argv[2], "-s") == 0)
        {
            shortcut_use(argv[3]);
        }
    }
    else if (strcmp(argv[1], "checkout") == 0)
    {
        char *address = where_before();
        strcat(address, "/.neogit/staging");
        FILE *file = fopen(address, "r");
        char *temp = (char *)malloc(20 * sizeof(char));
        int flag = 0;

        if (fscanf(file, "%s", temp) != EOF)
        {
            printf("your staging area is not emtpy!");
            flag = 1;
        }

        if (flag == 0)
        {

            if (strcmp(argv[2], "HEAD") == 0 && argc == 3)
            {
                char *address = where_before();
                strcat(address, "/.neogit/commits");
                chdir(address);
                DIR *dir = opendir(".");
                struct dirent *entry;
                int counter = 0;
                while ((entry = readdir(dir)) != NULL)
                {
                    if (strcmp(entry->d_name, ".") != 0 && strcmp(entry->d_name, "..") != 0 && strcmp(entry->d_name, ".DS_Store") != 0)
                    {
                        counter++;
                    }
                }
                for (int i = 1; i <= counter; i++)
                {
                    char temp[5];
                    sprintf(temp, "%d", i);
                    apply_checkout(temp);
                }
                HEAD_status_to_zero();
            }
            else if (strcmp(argv[2], "HEAD") == 0 && argc == 5 && strcmp(argv[3], "-n") == 0)
            {
                char *address = where_before();
                strcat(address, "/.neogit/commits");
                chdir(address);
                DIR *dir = opendir(".");
                struct dirent *entry;
                int counter = 0;
                while ((entry = readdir(dir)) != NULL)
                {
                    if (strcmp(entry->d_name, ".") != 0 && strcmp(entry->d_name, "..") != 0 && strcmp(entry->d_name, ".DS_Store") != 0)
                    {
                        counter++;
                    }
                }
                int nu = atoi(argv[4]);
                int x = counter - nu;
                for (int i = 1; i <= x; i++)
                {
                    char temp[5];
                    sprintf(temp, "%d", i);
                    apply_checkout(temp);
                }
                if (nu == 0)
                {
                    HEAD_status_to_zero();
                }
                else
                {
                    HEAD_status_to_one();
                }
            }
            else
            {
                flag = 0;
                char *word = (char *)malloc(100 * sizeof(char));
                strcpy(word, argv[2]);
                for (int i = 0; i < strlen(word); i++)
                {
                    if (*(word + i) > '9' || *(word + i) < '0')
                    {
                        flag = 1;
                    }
                }
                if (flag == 1)
                {
                    checkout_to_branch(argv[2]);
                }
                else
                {
                    checkout_to_hash(argv[2]);
                }
            }
        }
    }
    else if (strcmp(argv[1], "alias") == 0)
    {
        if (strcmp(argv[2], "--global") == 0)
        {
            char *full_command = (char *)malloc(100 * sizeof(char));
            strcpy(full_command, argv[4]);
            strcat(full_command, " ");
            for (int i = 5; i < argc; i++)
            {
                strcat(full_command, argv[i]);
                strcat(full_command, " ");
            }
            global_alias(argv[3], full_command);
        }
        else
        {
            char *address = where_before();
            if (address != NULL)
            {
                chdir(address);
                chdir(".neogit");
                char *full_command = (char *)malloc(100 * sizeof(char));
                strcpy(full_command, argv[3]);
                strcat(full_command, " ");
                for (int i = 4; i < argc; i++)
                {
                    strcat(full_command, argv[i]);
                    strcat(full_command, " ");
                }
                FILE *file1 = fopen("alias", "r");
                FILE *file2 = fopen("t.txt", "w");
                char *temp = (char *)malloc(1000 * sizeof(char));
                while (fgets(temp, 1000, file1) != NULL)
                {
                    fprintf(file2, "%s", temp);
                }
                fprintf(file2, "%s: %s\n", argv[2], full_command);
                fclose(file2);
                free(temp);
                remove("alias");
                rename("t.txt", "alias");
            }
        }
    }
    else if (strcmp(argv[1], "log") == 0)
    {
        if (argc == 2)
            commits_log(-1);
        if (argc == 4 && strcmp(argv[2], "-n") == 0)
        {
            int n = (int)(argv[3][0] - '0');
            commits_log(n);
        }
        else if (argc == 4 && strcmp(argv[2], "-author") == 0)
        {
            author_log(argv[3]);
        }
        else if (strcmp(argv[2], "-since") == 0 || strcmp(argv[2], "-before") == 0)
        {
            time_log(argv[3], argv[2]);
        }
        else if (strcmp(argv[2], "-branch") == 0)
        {
            branch_log(argv[3]);
        }
        else if (strcmp(argv[2], "-search") == 0)
        {
            int a = how_many_commits();
            search_log(argv[3], 5);
            for (int i = a; i >= 1; i--)
            {
                for (int j = 3; j < argc; j++)
                {
                    search_log(argv[j], i);
                }
            }
        }
    }
    else if (strcmp(argv[1], "set") == 0)
    {
        if (strcmp(argv[2], "-m") == 0 && strcmp(argv[4], "-s") == 0)
        {
            char *address = where_before();
            char *temp = (char *)malloc(1000 * sizeof(char));
            strcpy(temp, address);
            strcat(temp, "/.neogit/shortcuts");
            FILE *file = fopen(temp, "a");
            fprintf(file, "%s: ", argv[5]);
            fprintf(file, "%s\n", argv[3]);
        }
        else
        {
            printf("enter a valid command");
        }
    }
    else if (strcmp(argv[1], "replace") == 0)
    {
        if (strcmp(argv[2], "-m") == 0 && strcmp(argv[4], "-s") == 0)
        {
            char *word = (char *)malloc(200 * sizeof(char));
            char *temp = (char *)malloc(200 * sizeof(char));
            temp = where_before();
            strcpy(word, argv[5]);
            strcat(word, ":");
            strcat(temp, "/.neogit/shortcuts");
            FILE *file = fopen(temp, "r");
            int flag = 0;
            temp = where_before();
            strcat(temp, "/.neogit/tmpp");
            FILE *file1 = fopen(temp, "w");
            char *tmp = (char *)malloc(100 * sizeof(char));
            while (fscanf(file, "%s", temp) != EOF)
            {
                if (strcmp(temp, word) == 0)
                {
                    flag = 1;
                    for (int i = 0; i < strlen(temp); i++)
                    {
                        if (*(temp + i) == '\n')
                        {
                            *(temp + i) = '\0';
                        }
                    }
                    fprintf(file1, "%s ", temp);
                    fprintf(file1, "%s\n", argv[3]);
                    fgets(temp, 1000, file);
                }
                else
                {
                    for (int i = 0; i < strlen(temp); i++)
                    {
                        if (*(temp + i) == '\n')
                        {
                            *(temp + i) = '\0';
                        }
                    }
                    fprintf(file1, "%s", temp);
                    fgets(temp, 1000, file);
                    for (int i = 0; i < strlen(temp); i++)
                    {
                        if (*(temp + i) == '\n')
                        {
                            *(temp + i) = '\0';
                        }
                    }
                    fprintf(file1, "%s\n", temp);
                }
            }
            if (flag == 0)
            {
                printf("this shortcut does not exist to be modified");
            }
            char *address = where_before();
            chdir(address);
            chdir(".neogit");
            remove("shortcuts");
            rename("tmpp", "shortcuts");
        }
        else
        {
            printf("enter a valid command");
        }
    }
    else if (strcmp(argv[1], "remove") == 0)
    {

        if (strcmp(argv[2], "-s") == 0)
        {
            char *word_to_find = (char *)malloc(100 * sizeof(char));
            strcpy(word_to_find, argv[3]);
            strcat(word_to_find, ":");
            char *address = (char *)malloc(200 * sizeof(char));
            address = where_before();
            strcat(address, "/.neogit/shortcuts");

            char *address2 = (char *)malloc(200 * sizeof(char));
            address2 = where_before();
            strcat(address2, "/.neogit/tmpp");

            FILE *file = fopen(address, "r");
            FILE *file2 = fopen(address2, "w");
            char *temp = (char *)malloc(100 * sizeof(char));
            int flag = 0;
            while (fscanf(file, "%s", temp) != EOF)
            {
                if (strcmp(temp, word_to_find) != 0)
                {
                    for (int i = 0; i < strlen(temp); i++)
                    {
                        if (*(temp + i) == '\n')
                        {
                            *(temp + i) = '\0';
                        }
                    }
                    fprintf(file2, "%s", temp);
                    fgets(temp, 100, file);
                    for (int i = 0; i < strlen(temp); i++)
                    {
                        if (*(temp + i) == '\n')
                        {
                            *(temp + i) = '\0';
                        }
                    }
                    fprintf(file2, "%s\n", temp);
                    flag = 1;
                }
                else
                {
                    fgets(temp, 100, file);
                }
            }
            address2 = where_before();
            strcat(address2, "/.neogit");
            strcat(address2, "/shortcuts");
            remove(address2);
            address2 = where_before();
            strcat(address2, "/.neogit");
            chdir(address2);
            rename("tmpp", "shortcuts");

            if (flag == 0)
            {
                printf("this shortcut does not exist");
            }
        }
        else
        {
            printf("enter a valid command");
        }
    }
    else if (strcmp(argv[1], "branch") == 0)
    {
        if (argc == 3)
        {
            branch_maker(argv[2]);
        }
        else if (argc == 2)
        {
            branch_show();
        }
    }
    else if (strcmp(argv[1], "tree") == 0)
    {
        tree();
    }
    else if (strcmp(argv[1], "tag") == 0)
    {
        if (argc == 2)
        {
            tag_shower();
        }
        else if (strcmp(argv[2], "show") != 0)
        {
            tag_handler(argv, argc);
        }
        else
        {
            tag_log(argv[3]);
        }
    }
    else if (strcmp(argv[1], "diff") == 0)
    {
        if (argc == 5 && strcmp(argv[2], "-f") == 0)
        {
            file_changer(argv[3], "1");
            file_changer(argv[4], "2");
            diff(argv[3], argv[4]);
        }
        else if (argc == 8 && strcmp(argv[5], "-line") == 0)
        {
            int a = atoi(argv[6]);
            int b = atoi(argv[7]);
            diff_line(argv[3], argv[4], a, b);
        }
    }
    else if (strcmp(argv[1], "grep") == 0)
    {
        if (argc == 6 && strcmp(argv[2], "-f") == 0 && strcmp(argv[4], "-p") == 0)
        {
            grep(argv[3], argv[5]);
        }
        else if (argc == 7 && strcmp(argv[2], "-f") == 0 && strcmp(argv[4], "-p") == 0 && strcmp(argv[6], "-n") == 0)
        {
            grep_n(argv[3], argv[5]);
        }
        else if (argc == 6 && strcmp(argv[2], "-c") == 0 && strcmp(argv[4], "-p") == 0)
        {
            int commit_id = atoi(argv[3]);
            grep_commit(commit_id, argv[5]);
        }
        else if (argc == 7 && strcmp(argv[2], "-c") == 0 && strcmp(argv[4], "-p") == 0 && strcmp(argv[6], "-n") == 0)
        {
            int commit_id = atoi(argv[3]);
            grep_commit_n(commit_id, argv[5]);
        }
    }
    else if (strcmp(argv[1], "pre-commit") == 0)
    {
        if (argc == 4 && strcmp(argv[2], "hooks") == 0 && strcmp(argv[3], "list") == 0)
        {
            printf("all hooks:\n");
            printf("1. todo-check\n");
            printf("2. eof-blank-space\n");
            printf("3. format-check\n");
            printf("4. balance-braces\n");
            printf("5. indentation-check\n");
            printf("6. static-error-check\n");
            printf("7. file-size-check\n");
            printf("8. character-limit\n");
            printf("9. time-limit\n");
        }
        else if (argc == 4 && strcmp(argv[2], "applied") == 0 && strcmp(argv[3], "hooks") == 0)
        {
            char *address = where_before();
            chdir(address);
            chdir(".neogit");
            FILE *file = fopen("precommit", "r");
            while (fgets(address, 1000, file))
            {
                if (*(address + strlen(address) - 1) == '\n')
                {
                    *(address + strlen(address) - 1) = '\0';
                }
                printf("%s\n", address);
            }
            fclose(file);
        }
        else if (argc == 5 && strcmp(argv[2], "add") == 0 && strcmp(argv[3], "hook") == 0)
        {
            char *address = where_before();
            chdir(address);
            chdir(".neogit");
            FILE *file = fopen("precommit", "a");
            fprintf(file, "%s\n", argv[4]);
            fclose(file);
        }
        else if (argc == 5 && strcmp(argv[2], "remove") == 0 && strcmp(argv[3], "hook") == 0)
        {
            char *address = where_before();
            chdir(address);
            chdir(".neogit");
            FILE *file1 = fopen("precommit", "r");
            FILE *file2 = fopen("tpp", "w");
            while (fgets(address, 100, file1) != NULL)
            {
                if (*(address + strlen(address) - 1) == '\n')
                {
                    *(address + strlen(address) - 1) = '\0';
                }
                if (strcmp(address, argv[4]) == 0)
                {
                    continue;
                }
                fprintf(file2, "%s\n", address);
            }
            fclose(file1);
            fclose(file2);
            remove("precommit");
            rename("tpp", "precommit");
        }
        else if (argc == 2)
        {
            pre_commit();
        }
    }
    // else
    // {
    //     char *address = where_before();
    //     strcat(address, "/.neogit/alias");
    //     FILE *file = fopen(address, "r");
    //     char *word = (char *)malloc(100 * sizeof(char));
    //     char *target = (char *)malloc(100 * sizeof(char));
    //     strcpy(target, argv[1]);
    //     strcat(target, ":");
    //     char *rest = (char *)malloc(1000 * sizeof(char));
    //     while (fscanf(file, "%s", word) != EOF)
    //     {
    //         fgets(rest, 1000, file);
    //         if (strcmp(word, target) == 0)
    //             ;
    //         {
    //             char *command = (char *)malloc(100 * sizeof(char));
    //             strcat(command, "neogit");
    //             strcat(command, rest);
    //             for (int i = 2; i < argc; i++)
    //             {
    //                 strcpy(rest, argv[i]);
    //                 strcat(command, " ");
    //                 strcat(command, argv[i]);
    //                 printf("argv[i] = %s\n", rest);
    //                 printf("=%s=", command);
    //                 printf("//%c//", argv[i][0]);
    //             }
    //             // printf("command = %s\n", command);
    //             system(command);
    //             break;
    //         }
    //     }
    // }

    return 0;
}
