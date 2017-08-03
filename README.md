# Qt-Programming
programming using C++ language

#include "speedometer.h"
#include "ui_speedometer.h"
#include <QSerialPort>
#include <QSerialPortInfo>
#include <QDebug>
#include <QWidget>
#include <QMessageBox>
#include "speedparameter.h"
#include "testinstruction.h"
#include <QFile>
//#include <QtTest/QTest>
#include <QThread>
#include <QStringListIterator>


Speedometer::Speedometer(QWidget *parent) :
    QDialog(parent),
    ui(new Ui::Speedometer)
{
//--------------initialize Arduino connection--------------------
    arduino_is_available = false;
    arduino_port_name = "";
    ui->setupUi(this);
//        selectspeed = 2; ///First 3 profiles
    arduino = new QSerialPort;
    serialBuffer = "";

   qDebug()<<"Number of available ports:" << QSerialPortInfo::availablePorts().length();
    foreach(const QSerialPortInfo &serialPortInfo, QSerialPortInfo::availablePorts()){
        qDebug()<<"Has Vendor ID:" << serialPortInfo.hasVendorIdentifier();
        if(serialPortInfo.hasVendorIdentifier()){
            qDebug()<<"Vendor ID:"<< serialPortInfo.vendorIdentifier();
        }
            qDebug()<<"Has Product ID:" << serialPortInfo.hasProductIdentifier();
            if(serialPortInfo.hasProductIdentifier()){
                qDebug()<<"Product ID:"<< serialPortInfo.productIdentifier();
            }
    }

    foreach(const QSerialPortInfo &serialPortInfo, QSerialPortInfo::availablePorts()){
        if(serialPortInfo.hasVendorIdentifier() && serialPortInfo.hasProductIdentifier()){
            if(serialPortInfo.vendorIdentifier() == arduino_uno_vendor_id){
                if(serialPortInfo.productIdentifier() == arduino_uno_product_id){
                    arduino_port_name = serialPortInfo.portName();
                    arduino_is_available =  true;
                }
            }
        }
    }

        if(arduino_is_available){
            //open and configure the serialport

            arduino->setPortName(arduino_port_name);
            arduino->open(QSerialPort::ReadWrite);
            arduino->setBaudRate(QSerialPort::Baud9600);
            arduino->setDataBits(QSerialPort::Data8);
            arduino->setParity(QSerialPort::NoParity);
            arduino->setStopBits(QSerialPort::OneStop);
            arduino->setFlowControl(QSerialPort::NoFlowControl);
            //QObject::connect(arduino,SIGNAL(readyRead()),this,SLOT(readSerial()));

        }else{
            //show error message
            qDebug()<<" Port Error, Couldn't find the Arduino' !";

         //   QMessageBox::warning(this, "Port Error, Couldn't find the Arduino !");
        }
//-----------------complete arduino initialize connection-----------------

//------------------------design layout for Speedometer gauge GUI------------------
    mSpeedGauge = new QcGaugeWidget;
    mSpeedGauge2 = new QcGaugeWidget;


    mSpeedGauge->addBackground(99);
    mSpeedGauge2->addBackground(99);


    QcBackgroundItem *bkg1 = mSpeedGauge->addBackground(92);
    QcBackgroundItem *bkg3 = mSpeedGauge2->addBackground(92);


    bkg1->clearrColors();
    bkg3->clearrColors();


    bkg1->addColor(0.1,Qt::black);
    bkg3->addColor(0.1,Qt::black);


    bkg1->addColor(1.0,Qt::white);
    bkg3->addColor(1.0,Qt::black);


    QcBackgroundItem *bkg2 = mSpeedGauge->addBackground(88);
    QcBackgroundItem *bkg4 = mSpeedGauge2->addBackground(88);


    bkg2->clearrColors();
    bkg4->clearrColors();


    bkg2->addColor(0.1,Qt::gray);
    bkg4->addColor(0.1,Qt::gray);


    bkg2->addColor(1.0,Qt::darkGray);
    bkg4->addColor(1.0,Qt::darkGray);


    mSpeedGauge->addArc(55);
    mSpeedGauge2->addArc(55);


    mSpeedGauge->addDegrees(65)->setValueRange(0,80);
    mSpeedGauge2->addDegrees(65)->setValueRange(0,80);




    mSpeedGauge->addValues(80)->setValueRange(0,160);
    mSpeedGauge2->addValues(80)->setValueRange(0,40);


    mSpeedGauge->addLabel(70)->setText("Km/h");
    mSpeedGauge2->addLabel(70)->setText("x100 r/min");


    QcLabelItem *lab = mSpeedGauge->addLabel(40);
    QcLabelItem *lab2 = mSpeedGauge2->addLabel(40);


    lab->setText("0");
    lab2->setText("0");


    mSpeedNeedle = mSpeedGauge->addNeedle(60);
    mSpeedNeedle2 = mSpeedGauge2->addNeedle(60);


    mSpeedNeedle->setLabel(lab);
    mSpeedNeedle2->setLabel(lab2);


    mSpeedNeedle->setColor(Qt::white);
    mSpeedNeedle2->setColor(Qt::white);


    mSpeedNeedle->setValueRange(0,160);
    mSpeedNeedle2->setValueRange(0,40);

    mSpeedGauge->addBackground(7);
    mSpeedGauge2->addBackground(7);

    mSpeedGauge->addGlass(88);
    mSpeedGauge2->addGlass(88);

    ui->verticalLayout->addWidget(mSpeedGauge);

    ui->pushButton->setText("Start");
    ui->pushButton_2->setText("Close");


    QString Image1,Image2,Image3,Image4;

    Image1 = (":/resources/SP1.png");
    Image2 = (":/resources/SP2.png");
    Image3 = (":/resources/SP3.png");
    Image4 = (":/resources/SP4.png");



    ui->label->setPixmap(Image1);


    ui->label_2->setPixmap(Image2);


    ui->label_3->setPixmap(Image3);


    ui->label_4->setPixmap(Image4);
}

Speedometer::~Speedometer()
{
    if(arduino->isOpen()){
        arduino->close();
    }
    delete ui;

}

void Speedometer::on_pushButton_clicked()
{
    timer1 = new QTimer(this);
    connect(timer1, SIGNAL(timeout()),this, SLOT(increaseValue()));

    timer1->start(20);
    timer2->start(20);
    mValue1 = 0;
    mValue2 = 0;
    mValue3 = 0;
    mValue4 = 0;
    mValue5 = 0;
    mValue6 = 0;
    mValue7 = 0;
    mValue8 = 0;
    mCounter1 = 0;
    mCounter2 = 0;
    mCounter3 = 0;
    mCounter4 = 0;
    mCounter5 = 0;
    maxspeed = 80;
    slopelinear = 0.5;
    maxrpm = 80;

    if (ui->radioButton_4->isChecked()){
    q1 = 300; q2= 380; q3= 500; q4= 600; q5= 680; q6 = 800; q7 = 880; q8 = 1000;}
    if (ui->radioButton->isChecked() || ui->radioButton_2->isChecked() || ui->radioButton_3->isChecked())
    {  q1 = 300; ///First 3 profiles
         q2 = 380; ///First 3 profiles
         q3 = 680; ///First 3 profiles
    }
}
void Speedometer::increaseValue()
{

        if (mCounter1<= q1 )
            selectspeed =1;

        if (mCounter1 >= q1 && mCounter1 <= q2 )
            selectspeed =2;

        if (mCounter1 >= q2 && mCounter1 <= q3 )
            selectspeed =3;

        if (mCounter1 > q3 && (ui->radioButton->isChecked() || ui->radioButton_2->isChecked() || ui->radioButton_3->isChecked()) )
            mCounter1 = 0;

        if (mCounter1 >= q3 && mCounter1 <= q4 && ui->radioButton_4->isChecked())
            selectspeed =4;

        if (mCounter1 >= q4 && mCounter1 <= q5 && ui->radioButton_4->isChecked())
            selectspeed =5;

        if (mCounter1 >= q5 && mCounter1 <= q6 && ui->radioButton_4->isChecked())
            selectspeed =6;

        if (mCounter1 >= q6 && mCounter1 <= q7 && ui->radioButton_4->isChecked())
            selectspeed =7;

        if (mCounter1 >= q7 && mCounter1 <= q8 && ui->radioButton_4->isChecked())
            selectspeed =8;

        if (mCounter1 > q8 && ui->radioButton_4->isChecked() )
            mCounter1 = 0;

        switch (selectspeed){

        case 1:
      //  mValue1 = ( maxspeed*0.5*(tanh(0.053*mCounter1-1)+1) );
        if (ui->radioButton->isChecked()){ //SP1
        mValue1 = 2.67*mCounter1*0.1; }///First profile

        if (ui->radioButton_2->isChecked()){ //SP2
        mValue1 = 1.33*mCounter1*0.1;} ///Second profile

        if (ui->radioButton_3->isChecked()){
        mValue1 = 2.67*mCounter1*0.1;} ///Third profile

        if (ui->radioButton_4->isChecked()){
        mValue1 = 2*mCounter1*0.1;}
        mSpeedNeedle->setCurrentValue(mValue1);
        mCounter1++;
        mCounter2 = 0;
        mCounter3 = 0;
        mCounter4 = 0;
        mCounter5 = 0;
        break;
    case 2:

        if (ui->radioButton->isChecked()){ //SP1
        mValue2 = maxspeed; }///First Profile

        if (ui->radioButton_2->isChecked()){ //SP2
        mValue2 = maxspeed/2;}///Second Profile

        if (ui->radioButton_3->isChecked()){
        mCounter3++; ///For the 3rd profile
        mValue2 = mValue1 - 0.5*mCounter3;} ///Third Profile

        if (ui->radioButton_4->isChecked()){
        mValue2 = maxspeed*0.75;}

        mSpeedNeedle->setCurrentValue(mValue2);
        mCounter1++;

        break;
    case 3:

       if (ui->radioButton->isChecked()){ //SP1

       mValue3 = mValue2 -2.67*mCounter2*0.1;} /// First Profile

       if (ui->radioButton_2->isChecked()){
       mValue3 = mValue2 + 1.33*mCounter2*0.1;}  /// Second Profile

       if (ui->radioButton_3->isChecked()){
       mValue3 = mValue2 + 1.33*mCounter2*0.1;}/// Third Profile

       if (ui->radioButton_4->isChecked()){
       mValue3 = mValue2+ 1.67*mCounter2*0.1;}

        mSpeedNeedle->setCurrentValue(mValue3);
        mCounter1++;
        mCounter2++;
        break;
    case 4:

        mValue4 = mValue3 - 3*mCounter3*0.1;
        mSpeedNeedle->setCurrentValue(mValue4);
        mCounter1++;
        mCounter3++;

        break;
    case 5:

        mValue5 = maxspeed*0.625;
        mCounter1++;
        mSpeedNeedle->setCurrentValue(mValue5);
        break;
    case 6:

        mValue6 = mValue5 + 1.67*mCounter4*0.1;
        mCounter1++;
        mCounter4++;
        mSpeedNeedle->setCurrentValue(mValue6);
        break;
    case 7:

        mValue7 = 0.875*maxspeed;
        mCounter1++;
        mSpeedNeedle->setCurrentValue(mValue7);

        break;
    case 8:

        mValue8 = mValue7 - 6.67*mCounter5*0.1;
        mCounter1++;
        mCounter5++;
        mSpeedNeedle->setCurrentValue(mValue8);
        break;
    default:
    mCounter1++;
        break;
    }

}
---------------------end of design layout of Speedometer GUI----------------
void Speedometer::on_pushButton_3_clicked()
{
    timer1->stop();
    timer2->stop();
   
}


void Speedometer::updateSpeedometer(QString command)
{
    qDebug()<<"In Arduino Loop";
    if(arduino->isWritable())

    {
        qDebug()<<"command :"<<command;
        command.remove(" ");
        qDebug()<<"command after remove :"<<command;
        arduino->write(command.toStdString().c_str());
        qDebug() << command.toStdString().c_str() << " is uploaded to Arduino";
           }else{
        qDebug()<<"Couldn't write to Serial !" ;
    }
        qDebug()<<"Out Arduino Loop";
}


void Speedometer::on_SpeedometerSlider_valueChanged(int value)
{
    ui->SpeedoLabel->setText(QString("<span style-\" font-size:18pt;font-weight:600;\">%1</span>").arg(value));
    qDebug()<<value;
    Speedometer::updateSpeedometer(QString("s%1").arg(value));
    mSpeedNeedle->setCurrentValue(value*0.2260589448698748198592783068185);
}

void Speedometer::on_pushButton_4_clicked() //LOAD TEXT LINE BY LINE HERE
{
    qDebug() <<"Pushbutton clicked";
    ui->textEdit->clear();
           //QFile testinst("G:/HANDSHAKE TRIAL/ATE 17012017/testinstruction.txt");
    QFile readalltestinst("C:/Users/WAN SHAHMISUFI/Google Drive/00 from HP-Crest 31032017/CREST/ATE 17012017/testinstruction.txt");

     if (readalltestinst.open(QIODevice::ReadOnly)||QIODevice::Text)
    {
        QTextStream readall(&readalltestinst);
           QString content = readall.readAll();
           qDebug() <<"What is inside the text file? ---> "<< content;

               ui->textEdit->setText(content);
               QRegExp rx("s(\\d+)"); //<<--------------QREGEXP for number extract
               QList<int> list;
               int pos = 0;
               while ((pos = rx.indexIn(content, pos)) != -1) {
                   list << rx.cap(1).toInt();
                   pos += rx.matchedLength();
               }
               qDebug()<<"What's the speed inside this text file?-->"<<list;//<<-----------------------Display the number in term of QList

               QRegExp rx1("d(\\d+)"); //<<--------------QREGEXP for number extract
               QList<int> list1;
               int pos1 = 0;
               while ((pos1 = rx1.indexIn(content, pos1)) != -1) {
                   list1 << rx1.cap(1).toInt();
                   pos1 += rx1.matchedLength();
               }
               qDebug()<<"What's the duration inside this text file?-->"<<list1;//<<-----------------------Display the number in term of QList


               while(!list.empty())
               {
                  //int countLine = 0;
                  qDebug()<<"The first line of speed ="<<list[0];
                  qDebug()<<"The first line of duration ="<<list1[0];
                  int listFirst = list[0];
                  qDebug()<<"listFirst ="<<listFirst;
                  QString countString = QString::number(listFirst);
                  qDebug()<<"countString = "<<countString;

                  updateSpeedometer(QString("s%1").arg(listFirst));

                  qDebug()<<"Delay for = "<<0.7*list1[0] << " milliseconds";
                  QThread::msleep(0.7*list1[0]);

                  //Speedometer::createStartFile();

                  qDebug()<<"Delay for = "<<0.3*list1[0] << " milliseconds";
                  QThread::msleep(0.3*list1[0]);
                 // readSerial();
                  list.removeFirst();
                  list1.removeFirst();
                 
               }
    }

    qDebug() <<"Out Pushbutton File";
}

}

void Speedometer::ArduinoToGauge(const QString SpeedFreq)
{
    //mSpeedNeedle->setCurrentValue(SpeedFreq);
    int convert = SpeedFreq.toInt();
    mSpeedNeedle->setCurrentValue(convert*(0.2260589448698748198592783068185));
}
