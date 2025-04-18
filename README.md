# Virtual_memory_stimulation.c
#include <stdio.h>
#include <stdlib.h>

#define PAGE_SIZE 256
#define NUM_PAGES 8
#define NUM_FRAMES 4
#define MEMORY_SIZE (NUM_FRAMES * PAGE_SIZE)

int page_table[NUM_PAGES];
int frame_queue[NUM_FRAMES];
int memory[MEMORY_SIZE];

int front = 0, rear = -1, count = 0;
int frame_index = 0;
int page_faults = 0;

void init() {
    for (int i = 0; i < NUM_PAGES; i++)
        page_table[i] = -1;

    for (int i = 0; i < MEMORY_SIZE; i++)
        memory[i] = 0;
}

int fifo_replace(int page_num) {
    int replaced_page = frame_queue[front];
    page_table[replaced_page] = -1;

    front = (front + 1) % NUM_FRAMES;
    count--;

    return page_table[page_num];
}

void load_page(int page_num) {
    if (count < NUM_FRAMES) {
        frame_index = count;
        rear = (rear + 1) % NUM_FRAMES;
        frame_queue[rear] = page_num;
        count++;
    } else {
        int victim = frame_queue[front];
        page_table[victim] = -1;
        frame_index = page_table[victim];
        frame_queue[front] = page_num;
        front = (front + 1) % NUM_FRAMES;
    }

    page_table[page_num] = frame_index;
    page_faults++;

    printf("Page %d loaded into frame %d\n", page_num, frame_index);
}

void access_memory(int virtual_address) {
    int page_num = virtual_address / PAGE_SIZE;
    int offset = virtual_address % PAGE_SIZE;

    printf("\nAccessing virtual address: %d (Page: %d, Offset: %d)\n", virtual_address, page_num, offset);

    if (page_table[page_num] == -1) {
        printf("Page fault! Page %d not in memory.\n", page_num);
        load_page(page_num);
    } else {
        printf("Page %d is already in frame %d.\n", page_num, page_table[page_num]);
    }

    int physical_address = page_table[page_num] * PAGE_SIZE + offset;
    printf("Translated to physical address: %d\n", physical_address);
}

int main() {
    init();

    int accesses[] = {100, 300, 500, 700, 900, 300, 100, 1024};
    int n = sizeof(accesses) / sizeof(accesses[0]);

    for (int i = 0; i < n; i++) {
        access_memory(accesses[i]);
    }

    printf("\nTotal page faults: %d\n", page_faults);
    return 0;
}
