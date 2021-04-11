---
layout: post
title:  "验证xml是否符合指定xsd"
date:   2018-01-23 23:23:17
categories: .NET
tags: C# XSD XML
author: Michael
mathjax: true
---

* content
{:toc}

xml是常用的一种数据文件格式，它的定义文件为Xml schema definition（XSD）,那么怎么验证一个xml是否符合它的schema定义呢?
本文给出C#的代码实现。



### 样例XML
存储在xml.xml文件中
```xml
<?xml version="1.0" encoding="utf-8" ?>
<xml>
  <age>10</age>
  <date>2018-01-01</date>
  <regex>111</regex>
  <gMonth>---10--</gMonth>
  <language>english</language>
  <anyURI>./news.html</anyURI>
</xml>
```
### 样例xsd
存储在xsd.xsd文件中
```xml
<?xml version="1.0"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema">

  <xs:element name="xml">
    <xs:complexType>
      <xs:sequence>
        <xs:element name="age">
          <xs:simpleType>
            <xs:restriction base="xs:decimal">
              <xs:enumeration value="10"/>
              <xs:enumeration value="20"/>
              <xs:enumeration value="30"/>
            </xs:restriction>
          </xs:simpleType>
        </xs:element>
        <xs:element name="date">
          <xs:simpleType>
            <xs:restriction base="xs:date">
              <xs:enumeration value="2018-01-01Z"/>
            </xs:restriction>
          </xs:simpleType>
        </xs:element>
        <xs:element name="regex">
          <xs:simpleType>
            <xs:restriction base="xs:string">
              <xs:pattern value="[0-9][0-9][0-9]"/>
            </xs:restriction>
          </xs:simpleType>
        </xs:element>
        <xs:element name="gMonth" type="xs:gMonth"/>
        <xs:element name="language" type="xs:language"/>
        <xs:element name="anyURI" type="xs:anyURI"/>
      </xs:sequence>
    </xs:complexType>
  </xs:element>
</xs:schema>
```
### 完整代码
下面代码在Console Project中实现
```cs
class Program
    {
        static void Main(string[] args)
        {
            string xmlFile = "xml.xml";
            string xsdFile = "xsd.xsd";

            var exceptionMessage = string.Empty;
            VerifyXML(xsdFile, xmlFile, ref exceptionMessage);

            Console.WriteLine(exceptionMessage);
            Console.ReadKey();
        }

        private static void VerifyXML(string xsdFile, string xmlFile, ref string exceptionMessage)
        {
            XmlDocument doc = LoadXML(xmlFile);
            doc.Schemas = LoadXMLSchmeaFromXSDFile(xsdFile);

            string errorMessage = string.Empty;
            ValidationEventHandler eventHandler = new ValidationEventHandler(delegate (object sender, ValidationEventArgs e)
            {
                switch (e.Severity)
                {
                    case XmlSeverityType.Error:
                        errorMessage += e.Message;
                        break;
                    case XmlSeverityType.Warning:
                        break;
                }
            });

            doc.Validate(eventHandler);

            exceptionMessage = errorMessage;
        }

        private static XmlDocument LoadXML(string xmlFile)
        {
            XmlDocument doc = new XmlDocument();
            doc.Load(xmlFile);
            return doc;
        }

        private static XmlSchemaSet LoadXMLSchmeaFromXSDFile(string path)
        { 
            var schemas = new XmlSchemaSet();
            schemas.Add("", XmlReader.Create(path));
            return schemas;
        }
    }
```