//Author: Caroline de Freitas
//Date: 06/01/2019
//File name: before-update-field.groovy
//Description: Essa Script foi criado com o intuito de atualizar informações já existentes em um lead, para que a mudança ocorra no relatório 'Tabela de Preços'


// variáveis que serão atualizadas no relatório 
def upEstado = nvl (Estado_c, '');
def upCanal = nvl (Canal_c, '');
def upSubCanal = nvl (Segmento_c, '');
def bu = nvl (BusinessUnitName, '');

if (isAttributeChanged('BusinessUnitName')) {
  throw new oracle.jbo.ValidationException (' Você não tem permissão para alterar a Unidade de Negócio. ');
}

if (isAttributeChanged('Estado_c') && !Estado_c) {
   throw new oracle.jbo.ValidationException (' Por favor informe o Estado. ');
} 

if (isAttributeChanged('Canal_c') && !Canal_c) {
   throw new oracle.jbo.ValidationException (' Por favor informe o Canal. ');
}  

if (isAttributeChanged('Segmento_c') && !Segmento_c) {
   throw new oracle.jbo.ValidationException (' Por favor informe o Segmento. ');
}

// se não alterou os campos que afetam os preços, sai do trigger
if(isAttributeChanged("Estado_c") || isAttributeChanged("Canal_c") || isAttributeChanged("Segmento_c")) {
    def descTotal = 0.0;
    def descCanal = 0.0;
    def descEstado = 0.0;
    def descSegmento = 0.0;
    def tabDescEstado
    def tabDescontos
    def descontoProduto

    // vai buscar os descontos
    tabDescEstado = adf.util.pegaTabDescEstado(upEstado,bu)
    if(tabDescEstado != null) descEstado = nvl(tabDescEstado.Desconto_c, 0.0)

    tabDescontos = adf.util.pegaTabDescontos(bu,upCanal,upSubCanal)
    if(tabDescontos != null){
        descSegmento = nvl(tabDescontos.DescSegmento_c, 0.0)
        descCanal = nvl(tabDescontos.DescCanalSubCanal_c, 0.0)
    }
    descTotal += descEstado;
    descTotal += descSegmento;
    // termina a busca de desconto 
    
    //busca no mapa de Descontos
    Map<String,List> descontosProdutosMap = (Map<String,List>)BuscaMapaDescontos(upCanal);
    //busca no mapa de Preços
    Map<String,Double> precosProdutosMap = (Map<String,Double>)BuscaMapaPrecos(upEstado);
    
    // atualiza a linha de produto no lead
    def leadProds = MklProdAssoc1;
    
    while(leadProds.hasNext()){
        def prodUnit = leadProds.next();

        println (prodUnit.InventoryItemId);
        def prod = descontosProdutosMap.get(prodUnit.InventoryItemId.toString());
        if (prod!=null) {
            prodUnit.setAttribute('TabelaDescCanalProduto_Id_c', prod[0]);
            double descDif = (double)prod[1];
            descontoProduto = descTotal + descDif; // Desconto por Canal/Produto
            prodUnit.setAttribute('DescCanalDif_c',descDif.round(4));
        }else{
            descontoProduto = descTotal + descCanal; // Desconto por canal do cliente
        }
        
        def preco = precosProdutosMap.get(prodUnit.InventoryItemId.toString());
        if (preco!=null) {
            def precoComDesconto = Math.round(preco * (1 - descontoProduto) * 100 as Float)/100;
            prodUnit.setAttribute('Amount', precoComDesconto);
            println(prodUnit.InventoryItemId.toString()+": "+preco);
        }
    }
    println ('Processamento do Script finalizado');
}
