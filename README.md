# aws-step-fuctions-module
Projeto da unidade de Step Fuctions + Bedrock

# code:
{
  "Comment": "An example of using Bedrock to chain prompts and their responses together.",
  "StartAt": "Context and First Task Prompt",
  "States": {
    "Context and First Task Prompt": {
      "Type": "Task",
      "Resource": "arn:aws:states:::bedrock:invokeModel",
      "Parameters": {
        "ModelId": "arn:aws:bedrock:sa-east-1::foundation-model/anthropic.claude-3-haiku-20240307-v1:0",
        "Body": {
          "anthropic_version": "bedrock-2023-05-31",
          "max_tokens": 2000,
          "messages": [
            {
              "role": "user",
              "content": [
                {
                  "type": "text",
                  "text": "<context>Você um repórter especializado em críticas gastronômicas e em cervejas artesanais e é certificado como Bier Sommelier.</context><task>Sugira 3 lugares em Recife que são opções para viver uma experiência gastronômica que harmoniza alta gastronomia e cervejas</task><answer_settings>Forneça um pequeno resumo da essência do local e sugira um menu harmonizado com cerveja composto de entrada, prato princial e sobremesa, cada um harminozando com uma cerveja diferente. Explique a homonização em uma frase.</answer_settings>"
                }
              ]
            }
          ]
        },
        "ContentType": "application/json",
        "Accept": "*/*"
      },
      "Next": "Add first result to conversation history",
      "ResultPath": "$.result_one",
      "ResultSelector": {
        "result_one.$": "$.Body.content[0].text"
      }
    },
    "Add first result to conversation history": {
      "Type": "Pass",
      "Next": "Menu Suggestions Prompt",
      "Parameters": {
        "convo_one.$": "States.Format('{}\n{}', $.prompt_one, $.result_one.result_one)"
      },
      "ResultPath": "$.convo_one"
    },
    "Menu Suggestions Prompt": {
      "Type": "Task",
      "Resource": "arn:aws:states:::bedrock:invokeModel",
      "Parameters": {
        "ModelId": "arn:aws:bedrock:sa-east-1::foundation-model/anthropic.claude-3-haiku-20240307-v1:0",
        "Body": {
          "anthropic_version": "bedrock-2023-05-31",
          "max_tokens": 2000,
          "messages": [
            {
              "role": "user",
              "content": [
                {
                  "type": "text",
                  "text": "<task>Sugira 2 harmonizações de proteínas de pratos principais e formas de preparo para os seguintes tipos de cerveja: Bock, Weizen e Imperial Stout.</task><answer_settings>Forneça a lista com a sugestão acompanhada de uma explicação teórica da haronização em uma frase.</answer_settings>"
                }
              ]
            }
          ]
        },
        "ContentType": "application/json",
        "Accept": "*/*"
      },
      "Next": "Add second result to conversation history",
      "ResultSelector": {
        "result_two.$": "$.Body.content[0].text"
      },
      "ResultPath": "$.result_two"
    },
    "Add second result to conversation history": {
      "Type": "Pass",
      "Next": "Bier Styles Suggestion Prompt",
      "Parameters": {
        "convo_two.$": "States.Format('{}\n{}\n{}', $.convo_one.convo_one, $.prompt_two, $.result_two.result_two)"
      },
      "ResultPath": "$.convo_two"
    },
    "Bier Styles Suggestion Prompt": {
      "Type": "Task",
      "Resource": "arn:aws:states:::bedrock:invokeModel",
      "Parameters": {
        "ModelId": "arn:aws:bedrock:sa-east-1::foundation-model/anthropic.claude-3-haiku-20240307-v1:0",
        "Body": {
          "anthropic_version": "bedrock-2023-05-31",
          "max_tokens": 2000,
          "messages": [
            {
              "role": "user",
              "content": [
                {
                  "type": "text",
                  "text": "<task>Sugira 2 harmonizações de tipos de cerveja para os seguintes pratos: Cochinita Pibil, Leitão à Bairrada e Coelho à Caçadora.</task><answer_settings>Forneça a lista com a sugestão acompanhada de uma explicação teórica da haronização em uma frase.</answer_settings>"
                }
              ]
            }
          ]
        },
        "ContentType": "application/json",
        "Accept": "*/*"
      },
      "End": true,
      "ResultSelector": {
        "result_three.$": "$.Body.content[0].text"
      }
    }
  }
}
