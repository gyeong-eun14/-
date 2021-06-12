#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <math.h>
#pragma warning(disable:4996)
#define MAX_STACK_SIZE 100

typedef char element;// 스택 원소(element)의 자료형을 int로 정의 

typedef struct  stackNode {// 스택의 노드를 구조체로 정의
	element data[MAX_STACK_SIZE];
	int top;
} stackNode;


// 스택의 top 노드를 지정하기 위해 포인터 top 선언

void init_stack(stackNode* s)
{
	s->top = -1;
}

// 스택이 공백 상태인지 확인하는 연산
int isEmpty(stackNode* s)
{
	return (s->top == -1);
}

// 포화 상태 검출 함수
int is_full(stackNode* s)
{
	return (s->top == (MAX_STACK_SIZE - 1));
}

// 스택의 top에 원소를 삽입하는 연산
void push(stackNode* s, element item)
{
	if (is_full(s)) {
		fprintf(stderr, "스택 포화 에러\n");
		return;
	}
	else s->data[++(s->top)] = item;
}

// 스택의 top에서 원소를 삭제하는 연산
element pop(stackNode* s)
{
	if (isEmpty(s)) {
		fprintf(stderr, "스택 공백 에러\n");
		exit(1);
	}
	else return s->data[(s->top)--];
}

element peek(stackNode* s)
{
	if (isEmpty(s)) {
		fprintf(stderr, "스택 공백 에러\n");
		exit(1);
	}
	else return s->data[s->top];
}

int prec(char op)
{
	switch (op) {
	case '(': case ')': return 0;
	case '+': case '-': return 1;
	case '*': case '/': case '%': return 2;
	case '^': return 3;
	}
	return -1;
}

void infix_to_postfix(char exp[])
{

	int i = 0;
	char ch, top_op;
	int len = strlen(exp);

	stackNode s;
	init_stack(&s);// 스택 초기화 

	for (i = 0; i < len; i++) {
		ch = exp[i];
		switch (ch) {
		case '+': case '-': case '*': case '/': case '^': case '%': // 연산자
		// 스택에 있는 연산자의 우선순위가 더 크거나 같으면 출력
			while (!isEmpty(&s) && (prec(ch) <= prec(peek(&s))))
				printf("%c", pop(&s));
			push(&s, ch);
			break;
		case '(':// 왼쪽 괄호
			push(&s, ch);
			break;
		case ')':// 오른쪽 괄호
			top_op = pop(&s);
			// 왼쪽 괄호를 만날때까지 출력
			while (top_op != '(') {
				printf("%c", top_op);
				top_op = pop(&s);
			}
			break;
		default:// 피연산자
			printf("%c", ch);
			break;
		}
	}

	while (!isEmpty(&s))// 스택에 저장된 연산자들 출력
		printf("%c", pop(&s));
}

// 후위 표기법 수식을 계산하는 연산
int eval(char exp[])
{
	int op1, op2, value, i = 0;
	int length = strlen(exp);
	char symbol;
	stackNode s;

	init_stack(&s);
	for (i = 0; i < length; i++) {
		symbol = exp[i];
		if (symbol != '+' && symbol != '-' && symbol != '*' && symbol != '/' && symbol != '^' && symbol != '%') {
			value = symbol - '0';	// 입력이 피연산자이면
			push(&s, value);
		}
		else {	//연산자이면 피연산자를 스택에서 제거
			op2 = pop(&s);
			op1 = pop(&s);
			switch (symbol) { //연산을 수행하고 스택에 저장 
			case '+': push(&s, op1 + op2); break;
			case '-': push(&s, op1 - op2); break;
			case '*': push(&s, op1 * op2); break;
			case '/': push(&s, op1 / op2); break;
			case '^': push(&s, pow(op1,op2)); break;
			case '%': push(&s, op1 % op2); break;
			}
		}
	}
	return pop(&s);
}

// 수식 exp에 대한 처리를 마친 후 스택에 남아 있는 결과값을 pop하여 반환
void main(void) {
	int result;
	char* express;			// = "35*62/-";

	express = (char*)malloc(sizeof(char) * 100);
	printf("중위 표기식을 입력하시오: ");
	gets(express);

	printf("후위 표기식 : ");											//printf("후위 표기식 : %s\n", express);
	infix_to_postfix(express);

	result = eval(express);
	printf("\n연산 결과 => %d\n", result);

	free(express);
}
