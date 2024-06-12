#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_NAME_LEN 100
#define MAX_DOC_LEN 20

typedef struct {
    char origem[MAX_NAME_LEN];
    char destino[MAX_NAME_LEN];
    char data[MAX_NAME_LEN];
    char horario[MAX_NAME_LEN];
    int assentos_disponiveis;
    int id_voo;
} Voo;

typedef struct {
    char nome[MAX_NAME_LEN];
    char numero_documento[MAX_DOC_LEN];
} Passageiro;

typedef struct {
    int id_reserva;
    int id_voo;
    Passageiro passageiro;
} Reserva;

// adicionar voo
Voo* adicionar_voo(Voo *voos, int *num_voos, Voo novo_voo){
    voos = realloc(voos, (*num_voos + 1) * sizeof(Voo));
    if (voos == NULL){
        printf("Erro ao alocar memória para novos voos.\n");
        exit(1);
    }
    voos[*num_voos] = novo_voo;
    (*num_voos)++;
    return voos;
}

// listar voos
void listar_voos(Voo *voos, int num_voos){
    for (int i = 0; i < num_voos; i++){
        printf("Voo ID: %d, Origem: %s, Destino: %s, Data: %s, Horario: %s, Assentos disponíveis: %d\n",voos[i].id_voo, voos[i].origem, voos[i].destino, voos[i].data, voos[i].horario, voos[i].assentos_disponiveis);
    }
}

Voo* buscar_voos(Voo *voos, int num_voos, char *origem, char *destino, char *data, int *num_resultados){
    Voo *resultados = NULL;
    *num_resultados = 0;
    for (int i = 0; i < num_voos; i++){
        if (strcmp(voos[i].origem, origem) == 0 && strcmp(voos[i].destino, destino) == 0 && strcmp(voos[i].data, data) == 0){
            resultados = realloc(resultados, (*num_resultados + 1) * sizeof(Voo));
            if (resultados == NULL) {
                printf("Erro ao alocar memória para os resultados da busca.\n");
                exit(1);
            }
            resultados[*num_resultados] = voos[i];
            (*num_resultados)++;
        }
    }
    return resultados;
}

// Função para reservas de passagem
Reserva* adicionar_reserva(Reserva *reservas, int *num_reservas, Reserva nova_reserva){
    reservas = realloc(reservas, (*num_reservas + 1) * sizeof(Reserva));
    if (reservas == NULL){
        printf("Erro ao alocar memória para novas reservas.\n");
        exit(1);
    }
    reservas[*num_reservas] = nova_reserva;
    (*num_reservas)++;
    return reservas;
}

// listagem de reservas
void listar_reservas(Reserva *reservas, int num_reservas){
    for (int i = 0; i < num_reservas; i++){
        printf("Reserva ID: %d, Voo ID: %d, Nome do passageiro: %s, Documento: %s\n",reservas[i].id_reserva, reservas[i].id_voo, reservas[i].passageiro.nome, reservas[i].passageiro.numero_documento);
    }
}

// cancelamento das reservas
Reserva* cancelar_reserva(Reserva *reservas, int *num_reservas, int id_reserva){
    int index = -1;
    for (int i = 0; i < *num_reservas; i++){
        if (reservas[i].id_reserva == id_reserva){
            index = i;
            break;
        }
    }
    if (index != -1){
        for (int i = index; i < *num_reservas - 1; i++){
            reservas[i] = reservas[i + 1];
        }
        (*num_reservas)--;
        reservas = realloc(reservas, (*num_reservas) * sizeof(Reserva));
        if (*num_reservas > 0 && reservas == NULL) {
            printf("Erro ao realocar memória para reservas.\n");
            exit(1);
        }
    }
    return reservas;
}

// Agora que toda a estrutura foi montada é hora de fazer o menu de interação
void menu(){
    printf("Sistema de Reservas de Passagens Aéreas\n");
    printf("1. Consultar voos disponíveis\n");
    printf("2. Reservar passagem\n");
    printf("3. Cancelar reserva\n");
    printf("4. Consultar reservas\n");
    printf("5. Sair\n");
    printf("Escolha uma opção: ");
}

int main(){
    Voo *voos = NULL;
    int num_voos = 0;
    Reserva *reservas = NULL;
    int num_reservas = 0;
    int opcao, id_voo, id_reserva, num_resultados;
    char origem[MAX_NAME_LEN], destino[MAX_NAME_LEN], data[MAX_NAME_LEN];
    Reserva nova_reserva;
    Passageiro novo_passageiro;

    do{
        menu();
        scanf("%d", &opcao);
        switch (opcao){
            case 1:
                printf("Origem: ");
                scanf("%s", origem);
                printf("Destino: ");
                scanf("%s", destino);
                printf("Data (DD/MM/AAAA): ");
                scanf("%s", data);
                Voo *resultados = buscar_voos(voos, num_voos, origem, destino, data, &num_resultados);
                if (num_resultados > 0){
                    listar_voos(resultados, num_resultados);
                } else{
                    printf("Nenhum voo encontrado para os critérios informados.\n");
                }
                free(resultados);
                break;

            case 2:
                printf("ID do Voo: ");
                scanf("%d", &id_voo);
                printf("Nome: ");
                scanf("%s", novo_passageiro.nome);
                printf("Número de Documento: ");
                scanf("%s", novo_passageiro.numero_documento);
                nova_reserva.id_voo = id_voo;
                nova_reserva.passageiro = novo_passageiro;
                nova_reserva.id_reserva = num_reservas + 1;
                reservas = adicionar_reserva(reservas, &num_reservas, nova_reserva);
                printf("Reserva realizada com sucesso!\n");
                break;

            case 3:
                printf("ID da Reserva: ");
                scanf("%d", &id_reserva);
                reservas = cancelar_reserva(reservas, &num_reservas, id_reserva);
                printf("Reserva cancelada com sucesso.\n");
                break;

            case 4:
                listar_reservas(reservas, num_reservas);
                break;

            case 5:
                printf("Sair\n");
                break;

            default:
                printf("Opção inválida. Tente novamente.\n");
        }
    } while (opcao != 5);

    free(voos);
    free(reservas);
    
    return 0;
}
