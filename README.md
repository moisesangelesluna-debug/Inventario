import java.util.Scanner;
import java.util.ArrayList;
import java.util.List;
public class InventarioCalculadora {
private static float D;  // Demanda anual
private static float cP; // Costo por Pedido
private static float cA; // Costo de Almacenamiento Anual
private static float tE; // Tiempo de Entrega (dias)
private static float cO; // Cantidad Optima de Pedido (Q*)
private static float tP; // Tiempo entre Pedidos (años)
private static float le; // Punto de Reorden (unidades)
private static float TCU; // Costo Total Anual de Inventario
private static float D_diaria;
private static int n;    
public static void main(String[] args) {
    Scanner sc = new Scanner(System.in);
    int opcion;
    System.out.println(" MODULO DE INVENTARIO (EOQ) ");
    do {
        System.out.println("\nSeleccione el modelo:");
        System.out.println("1. Inventario sin descuento");
        System.out.println("2. Inventario con descuento por cantidad");
        try {
            opcion = sc.nextInt();
        } catch (Exception e) {
            opcion = 0;
            sc.next();
       }
       switch (opcion) {
           case 1:
                CalcularEOQBasico(sc);
                MostrarResultados();
                break;
           case 2:
                CalcularDescuentoPorCantidad(sc);
                break;
           case 3:
                System.out.println("Saliendo del programa...");
                break;
           default:
                System.out.println("Opcion no valida.");
                break;
       }     
    } while (opcion != 3);
    sc.close();
}
private static void PedirDatosBásicos(Scanner sc) {
    System.out.print("Ingrese la Demanda Anual: ");
    D = sc.nextFloat();
    System.out.print("Ingrese el Costo por Pedido: ");
    cP = sc.nextFloat();
    System.out.print("Ingrese el Costo de Almacenamiento Anual por Unidad: ");
    cA = sc.nextFloat();
    D_diaria = D / 365.0f;
    System.out.print("Ingrese el Tiempo de Entrega en DIAS: ");
    tE = sc.nextFloat();
}
private static void CalcularEOQBasico(Scanner sc) {
    System.out.println("\n CALCULO EOQ BASICO ");
    PedirDatosBásicos(sc);
    // 1. Cantidad Optima de Pedido
    cO = (float) Math.sqrt((2 * D * cP) / cA);
    // 2. Tiempo entre Pedidos en años
    tP = cO / D;
    // 3. Punto de Reorden
    le = D_diaria * tE;
    // 4. Costo Total de Inventario - Sin costo de compra 
    TCU = (cA * cO / 2) + (cP * (D / cO));
    // 5. Numero de Ciclos
    float tE_anual = tE / 365.0f;
    n = (int) Math.floor(tE_anual / tP);
}
private static void MostrarResultados() {
    System.out.println("\n*** RESULTADOS DEL CALCULO ***");
    System.out.printf("Cantidad Optima de Pedido: %.2f unidades%n", cO);
    System.out.printf("Tiempo entre Pedidos: %.2f dIas%n", tP * 365);
    System.out.printf("Punto de Reorden (le): %.0f unidades%n", le);
    System.out.printf("Costo Total Anual de Inventario: $%.2f%n", TCU);
    System.out.printf("Numero de ciclos de pedido (n): %d%n", n);
}
    
private static void CalcularDescuentoPorCantidad(Scanner sc) {
    System.out.println("\n CALCULO DESCUENTO POR CANTIDAD ");
    System.out.print("Ingrese la Demanda Anual: ");
    D = sc.nextFloat();
    System.out.print("Ingrese el Costo por Pedido: ");
    cP = sc.nextFloat();
    System.out.print("Ingrese el Factor de Costo de Almacenamiento como porcentaje (ej: 20 para 20%): ");
    float I_pct = sc.nextFloat();
    float I = I_pct / 100.0f;
    System.out.print("Ingrese el numero de rangos de precio: ");
    int numRangos = sc.nextInt();
    List<ResultadoRango> rangos = new ArrayList<>();
    for (int i = 0; i < numRangos; i++) {
        System.out.println("\n--- Rango " + (i + 1) + " ---");
        System.out.print("Cantidad MÍNIMA (Q_min): ");
        float Q_min_rango = sc.nextFloat();
        System.out.print("Costo Unitario (C): ");
        float C_rango = sc.nextFloat();
        float H = I * C_rango;
        // 1. Calcular el EOQ sin restriccion (Q_prima)
        float Q_prima = (float) Math.sqrt((2 * D * cP) / H);
        // 2. Ajustar la Q* al rango
        float Q_ajustada;
        if (Q_prima < Q_min_rango) {
            Q_ajustada = Q_min_rango;
        } else {
            Q_ajustada = Q_prima;
        }
        // 3. Calcular el Costo Total Anual
        float CC = D * C_rango; // Costo de Compra Anual
        float CO = cP * (D / Q_ajustada); // Costo de Pedido Anual
        float CM = H * (Q_ajustada / 2.0f); // Costo de Mantenimiento Anual
        float TCU_rango = CC + CO + CM;
        rangos.add(new ResultadoRango(Q_min_rango, C_rango, Q_ajustada, TCU_rango));
    }
    // --- Encontrar la mejor opcion ---    
    ResultadoRango mejorOpcion = rangos.get(0);
    for (ResultadoRango r : rangos) {
        if (r.TCU < mejorOpcion.TCU) {
            mejorOpcion = r;
        }
    }
    cO = mejorOpcion.Q_ajustada;
    TCU = mejorOpcion.TCU;
    D_diaria = D / 365.0f;
    tP = cO / D;
    System.out.print("\nIngrese el Tiempo de Entrega en dias para la mejor opcion: ");
    tE = sc.nextFloat();
    le = D_diaria * tE;
    float tE_anual = tE / 365.0f;
    n = (int) Math.floor(tE_anual / tP);
    System.out.println("\n*** MEJOR OPCION ENCONTRADA ***");
    System.out.printf("Rango elegido (Q_min): %.0f (Costo Unitario: $%.2f)%n", mejorOpcion.Q_min, mejorOpcion.C);
    System.out.printf("Cantidad Optima de Pedido (cO): %.2f unidades%n", cO);
    System.out.printf("Punto de Reorden (le): %.0f unidades%n", le);
    System.out.printf("Costo Total Anual Mínimo (TCU): $%.2f%n", TCU);
    System.out.printf("Tiempo entre Pedidos (tP): %.2f días%n", tP * 365);
}
private static class ResultadoRango {
    float Q_min;
    float C;
    float Q_ajustada;
    float TCU;
    public ResultadoRango(float Q_min, float C, float Q_ajustada, float TCU) {
        this.Q_min = Q_min;
        this.C = C;
        this.Q_ajustada = Q_ajustada;
        this.TCU = TCU;
     }   
}
}

    
       
